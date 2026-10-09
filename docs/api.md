# API (spike, v0)

The HTTP surface of Bowei's server. Object shapes are in
[`data-format.md`](data-format.md). This file has the product flows first (which
calls each screen makes, in order), then the endpoint reference.

## Basics

- **Base URL**: set in each extension's popup (`serverUrl`). Must be HTTPS:
  YouTube is HTTPS, so a plain-HTTP server's media would be blocked as mixed
  content.
- **Prefix**: every JSON route is under `/api/v0`. Media and the HTML pages are
  outside it.
- **Identity (spike stand-in for auth)**: each person in the hard-coded circle
  has a fixed token, pasted into the popup once. Every `/api/v0` call sends
  `Authorization: Bearer <token>`. Real sign-in can later swap how the token is
  obtained without changing any route.
- **CORS**: the server answers every origin (`Access-Control-Allow-Origin: *`,
  `Authorization` and the `X-Client*` headers allowed). No cookies are used, so
  this is safe, and content scripts can call the server directly on both Chrome
  and Firefox. Background-script calls work too. Use the background script for
  anything that should survive navigation (uploads, badge polling).
- **Debug headers**: send `X-Client` and `X-Client-Session` on every call
  (`data-format.md`, Debug metadata).
- **Errors**: non-2xx responses carry
  `{ "error": { "code": "media_missing", "message": "…", "requestId": "…" } }`.
  Codes: `unauthorized`, `not_found`, `forbidden`, `invalid`, `media_missing`,
  `id_conflict`, `too_large`.
- **Idempotent creates**: `PUT /…/{clientGeneratedId}`. Repeating a PUT with the
  same body returns `200` and the existing object. The same id with a different
  body returns `409 id_conflict`. Retry freely.

## Product flows

The demo loop from `interactions.md` (session 521), as calls. "Studio" is the
recording extension, "Feed" is the feed-buttons and overlay extension.

### 1. Set up (each person, once)

1. Open the popup, paste `serverUrl` and token.
2. `GET /api/v0/me` returns the person: show "Signed in as Bowei" and the
   server build in the popup. A wrong token shows the error right there.

### 2. Record and keep a reaction (Studio, watch page)

1. Record and retake locally. Nothing is sent.
2. On **Keep**:
   1. `PUT /api/v0/media/{mediaId}` with the recording bytes.
   2. Optional: `PUT /api/v0/media/{posterId}` with a JPEG frame.
   3. `PUT /api/v0/reactions/{reactionId}` with the reaction, the `video`
      object, and `capture`.
3. Show the progress-bar marker immediately (optimistic). If the upload fails,
   keep the take queued in IndexedDB, retry in the background, and post
   `upload.retry` telemetry.
4. Nudging the anchor after Keep: `PATCH /api/v0/reactions/{id}` with
   `anchorSec` and the full `anchorNudges` list.

### 3. Send to the circle (Studio, watch page)

1. Open the send sheet.
2. `GET /api/v0/people` gives the recipient chips, people you've sent to most
   recently first.
3. `GET /api/v0/videos/{ref}/reactions?author=me` gives your kept reactions on
   this video, all ticked.
4. **Send**: `PUT /api/v0/sends/{sendId}` with ticked `reactionIds`, `note`,
   `recipientIds`, and `origin: "watch_page"`.
5. The response lists the recipients: show "Sent to Wife".

If a reaction is still uploading, the sheet waits for its PUT to finish before
sending. The server rejects a send naming an unknown reaction.

### 4. Notice it (recipient, background + YouTube home)

1. Background script: `GET /api/v0/inbox/count` every 60 s. Show `unopened` as
   the extension badge.
2. YouTube home: `GET /api/v0/inbox` renders the **From your circle** shelf.
   Unopened sends come first, then newest. Each item already holds the video,
   the sender, the note, and the reactions, so the shelf makes no other calls.
3. **Dismiss** on a card: `POST /api/v0/sends/{id}/state` with
   `{ "state": "dismissed" }`.

### 5. Watch with reactions (anyone, watch page)

1. On every navigation to `/watch?v=ID`:
   `GET /api/v0/videos/yt:ID/reactions`. This returns every reaction the
   viewer can see: their own, plus everything sent to them. Each one carries its
   author, media URL, and the ids of the sends it arrived in. Draw a marker per
   reaction and run the sync loop (`data-format.md`, Playback meaning).
2. If the response's `inboxSendIds` is non-empty, post
   `{ "state": "opened" }` for each of those sends.
3. When every reaction in a send has finished or been skipped, or the base
   video passes 90% for a send with no reactions, post
   `{ "state": "watched" }` for that send.
4. Telemetry from this page: `watch.loaded`, `reaction.started` / `finished` /
   `skipped`, `drift_corrected`, `media_error`. Batch it and flush every 10 s
   and on `pagehide`.

### 6. React back (recipient, watch page)

1. Same recording flow as 2, with `replyToReactionId` set to the reaction being
   answered (the one on screen when React was pressed, if any).
2. After Keep, open the send sheet prefilled with the original sender ticked,
   `replyToSendId` set, and `origin: "react_back"`. One tap sends it.
3. The original sender now sees the reply on the same video (flow 5), as the
   replier's marker.

### 7. Send a pick (Feed, any video card or watch page)

1. The **Send** button on a card opens the send sheet with no reactions.
2. Read the `video` object from the card: id, title, channel. `durationSec` is
   usually unknown there; leave it null.
3. `PUT /api/v0/sends/{id}` with `reactionIds: []`, the note ("why you'd like
   this"), `recipientIds`, and `origin: "feed_card"`.
4. Optional "add a reaction": go to the watch page and run flow 2. Then send
   from there, which makes a second send. The spike doesn't merge them.

### 8. Get picks (Bowei, YouTube home)

Same as flow 4, on Bowei's browser. Watching a pick is flow 5, and reacting back
is flow 6. The loop is closed.

### 9. Creator link (stretch)

1. The send sheet gets a **Creator** chip, ticked by default, for the video's
   channel.
2. Sending with `creatorLink: true` makes the server mint a link, returned as
   `send.links[0].url`. Bowei pastes it into email or Twitter himself.
3. `GET /l/{token}` is a plain page: a YouTube embed, the send's reactions on a
   timeline under it playing at their moments, and a reply box.
4. Replies: `POST /api/v0/links/{token}/replies` (no bearer token; the link is
   the credential). They show up in Bowei's inbox as a send from a `creator`
   person the server creates for that channel.

Not built in the spike: rate limiting, link expiry, voice replies on the page.

## Endpoint reference

All bodies are JSON unless stated. "Me" is the person the bearer token names.

### Server and people

**`GET /api/v0/health`**: no auth. Returns
`{ "ok": true, "serverBuild": "…", "time": "…" }`.

**`GET /api/v0/me`**: returns `{ "person": Person, "serverBuild": "…" }`.

**`GET /api/v0/people`**: the circle, minus me.
`{ "people": [ Person & { "lastSentToAt": "…" | null } ] }`, sorted by
`lastSentToAt` descending, then by name.

### Media

**`PUT /api/v0/media/{mediaId}`**: the body is the raw file bytes, with
`Content-Type` set to the recorder's MIME type. Maximum 200 MB. Returns
`Media`: `201` when new, `200` on an identical repeat (same sha256).

**`GET /media/{mediaId}`**: no auth (capability URL). Serves the original bytes
with the stored `Content-Type`, `Accept-Ranges: bytes`, and `206` range
responses, so `<video src>` can seek. CORS `*`. Long cache headers: media is
immutable.

### Reactions

**`PUT /api/v0/reactions/{reactionId}`**:

```jsonc
{
  "video": { "ref": "yt:dQw4w9WgXcQ", "title": "…", "channelName": "…", "durationSec": 212.091 },
  "kind": "video",
  "syncMode": "playing",
  "anchorSec": 83.417,
  "durationSec": 11.84,
  "text": null,
  "mediaId": "6f1c…",
  "posterMediaId": null,
  "replyToReactionId": null,
  "capture": { /* CaptureInfo */ }
}
```

`authorId` is me. Validation:

- `mediaId` is required for `video`/`audio` and must already be uploaded by me
  (`409 media_missing` otherwise).
- `anchorSec` must be `>= 0` and, when the video's duration is known, no more
  than `durationSec + 1`.

Returns `Reaction` with `author` and `media` embedded.

**`PATCH /api/v0/reactions/{reactionId}`**: author only. The body may set
`anchorSec`, `syncMode`, `text`, and `capture.anchorNudges`. Returns `Reaction`.

**`DELETE /api/v0/reactions/{reactionId}`**: author only. Soft delete. The
reaction disappears from every read, including sends it was in. Returns `204`.

**`GET /api/v0/videos/{videoRef}/reactions`**: everything me can see on this
video (see visibility in `data-format.md`). Query: `author=me` or
`author=<personId>` filters by author. Returns:

```jsonc
{
  "video": Video,                   // null if the server has never seen it
  "reactions": [                    // sorted by anchorSec
    Reaction & {
      "author": Person, "media": Media | null, "posterMedia": Media | null,
      "viaSendIds": ["c41d…"]       // sends that showed it to me; [] for my own
    }
  ],
  "inboxSendIds": ["c41d…"]         // sends to me on this video that aren't dismissed (flow 5, step 2)
}
```

### Sends

**`PUT /api/v0/sends/{sendId}`**:

```jsonc
{
  "video": { "ref": "yt:…", "title": "…", "channelName": "…" },
  "reactionIds": ["a3e9…"],
  "note": "…",
  "recipientIds": ["wife"],
  "replyToSendId": null,
  "origin": "watch_page",
  "creatorLink": false,             // stretch, flow 9
  "client": { /* ClientInfo */ }
}
```

Validation:

- At least one recipient or `creatorLink`.
- Recipients must be in the circle and must not be me.
- Every reaction must be mine, on this video, and not deleted
  (`400 invalid` names the bad id).

Returns `Send` with `sender`, `video`, `reactions` and, when requested, `links`.

**`GET /api/v0/inbox`**: sends where I am a recipient. Query: `state` is
`active` (default: everything not dismissed), `unopened`, or `all` (includes
dismissed); `limit` defaults to 50. Returns:

```jsonc
{
  "items": [
    Send & {
      "sender": Person, "video": Video, "reactions": [ Reaction & { "author", "media", "posterMedia" } ],
      "me": { "openedAt": null, "watchedAt": null, "dismissedAt": null }   // my recipient state
    }
  ],
  "counts": { "unopened": 2, "total": 7 }
}
```

**`GET /api/v0/inbox/count`**: `{ "unopened": 2 }`. Cheap; for badge polling.

**`GET /api/v0/sent`**: my outgoing sends, newest first, with every recipient's
state. Use it for the "Sent to…" confirmation history, and for checking whether
she has opened it yet.

**`POST /api/v0/sends/{sendId}/state`**: recipient only.
`{ "state": "opened" | "watched" | "dismissed" }` sets the matching timestamp
for me, if not already set; repeats are no-ops. `watched` also sets `opened`.
Returns my recipient state.

### Telemetry

**`POST /api/v0/telemetry`**: a batch of client events (types in
`data-format.md`, Event log). `navigator.sendBeacon` works for the `pagehide`
flush: it can't set headers, so the endpoint also accepts the token as
`?token=…` and a `text/plain` body. Maximum 500 events per batch.

```jsonc
{ "events": [ { "type": "reaction.drift_corrected", "clientAt": "…", "videoRef": "yt:…", "reactionId": "…", "sendId": null, "data": { "driftSec": 0.42, "baseTimeSec": 88.1 } } ] }
```

Returns `{ "accepted": 37 }`.

### Debug

**`GET /debug`**: HTML, Bowei's token only (as `?token=`). Shows:

- people, videos, and reactions, with inline players and a capture-info diff
  (base-video advance vs. recording length);
- sends with each recipient's state;
- the last 500 requests and events, filterable by person, client session, and
  video.

Every row expands to its raw JSON.

**`GET /api/v0/debug/dump`**: Bowei only. Every table as one JSON document, for
backup or offline poking. Media are referenced by URL, not inlined.

## Client notes

- The two extensions keep separate settings, so each popup needs `serverUrl`
  and the token until they are merged into one extension.
- Build `ref` only as `"yt:" + videoId`. Shorts use the same id.
- Treat the server as the source of truth for kept reactions. IndexedDB holds
  drafts, the upload queue, and a read cache.
- Don't block the UI on telemetry. Drop events if the queue passes 1,000.

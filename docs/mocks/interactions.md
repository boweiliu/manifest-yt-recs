# Core-loop interactions (project 013)

Every interaction the build-night demo needs, in loop order. Sign-in, invites,
and permissions are out of scope: in the demo, everyone already has the
extension and is already "in". Mocks for each screen: `mocks.html` (one page,
frames numbered to match the sections below).

## People and places

- **Bowei**: owns the app. Reacts, sends, receives picks.
- **Friend** (the demo uses Bowei's wife): has the extension. Watches Bowei's
  reactions, sends picks back.
- **Creator** (stretch): has no extension. Gets a plain link.

Three surfaces:

1. **YouTube watch page + extension**: recording and playback of reactions.
2. **YouTube home page + extension**: the "From your circle" shelf, and a send
   button on each video card.
3. **Bowei's app** (a plain web page): the reaction viewer that a creator (or
   anyone without the extension) opens from a link. It also holds the raw data.

## One object: a "send"

A send is: a video, the sender's reactions on it (zero or more), a note ("why
you'd like this"), and recipients. Bowei sending his reaction to his wife and
his wife sending Bowei a pick are the same thing pointed in different
directions. That gives the demo one data shape, one send sheet, one shelf.

A reaction is: a video id, a moment (seconds), a kind (video, voice, text),
a play mode, and the media.

## 1. Record a reaction (Bowei, watch page)

1. Bowei watches a video. A **React** button sits in the player's control bar
   (keyboard: `R`).
2. Pressing it opens the **studio** docked beside the player, with the camera
   preview live. It captures the current moment as the reaction's anchor.
3. He picks the kind: video, voice, or text (video by default; the last choice
   sticks).
4. He picks the play mode:
   - **Over the video**: the base video keeps playing while he talks. The
     reaction later plays alongside at the same moment.
   - **On a paused frame**: the base video pauses on the frame. The reaction
     later pauses the viewer's video at that moment, plays, then resumes.
   Default: over the video. Pausing the base video by hand while the studio is
   open switches to paused-frame mode.
5. Record, stop. For text, he types instead; Enter saves.

## 2. Review the take (Bowei, watch page)

1. The studio switches to review: his recording plays back beside the frozen
   moment it is anchored to.
2. Speed: 1x, 1.5x, 2x.
3. Nudge the anchor (-1s / +1s) if he started late.
4. **Keep** or **Retake**. A kept reaction appears as a marker on the progress
   bar, with his avatar.
5. He can record more reactions on the same video. They pile up as markers.

## 3. Send to the circle (Bowei, watch page)

1. **Send** opens the send sheet, prefilled with every reaction he kept on this
   video (each one can be unticked).
2. Recipients are chips: people he has sent to before, plus the **creator**,
   ticked by default. One tap untick (this answers the "candor" open question
   per send, not per reaction).
3. An optional note.
4. **Send**. Confirmation shows who got it. The creator gets a link (frame 8).

## 4. Notice it (Friend, YouTube home)

1. The extension icon shows a count of unwatched sends.
2. YouTube home gets a **From your circle** shelf above the normal feed (it
   sits above rather than replacing, so the demo never fights YouTube's
   layout). Each card: thumbnail, the sender's face, their note, and how many
   reactions are on it.
3. Clicking a card opens the watch page normally.

## 5. Watch with reactions (Friend, watch page)

1. The progress bar shows the sender's markers (avatars) at each reaction.
2. At a marker's moment:
   - **Video reaction, over-the-video**: the sender's face appears in a bubble
     beside the player and plays in sync. If the friend pauses, it pauses.
   - **Paused-frame reaction**: the main video pauses on the frame and the
     reaction plays large over a dimmed player. It resumes on its own when the
     reaction ends; **Skip** resumes now.
   - **Voice**: the sender's avatar with a live waveform, same placement as
     video.
   - **Text**: a speech bubble beside the player for a few seconds.
3. Controls on the reaction bubble: mute, skip, and a reactions on/off toggle.
4. Clicking a marker jumps to that moment.
5. **React back**: the same React button. Her reply is anchored to the same
   video, so it shows up as her marker for Bowei.

## 6. Send a pick (Friend, YouTube home or watch page)

1. Every video card on YouTube gets a small **Send** button on hover (also in
   the watch page's action row).
2. It opens the same send sheet: recipients (Bowei, ticked), a "why you'd like
   it" note (the main ask), and an optional "add a reaction" that opens the
   studio on that video.
3. **Send**.

## 7. Get picks (Bowei, YouTube home)

1. Same **From your circle** shelf as frame 4, now on Bowei's YouTube. Each
   pick shows who sent it, their reason, and a reaction preview.
2. Watching a pick plays her reactions (frame 5). He reacts back (frame 1). The
   loop closes.

## 8. Creator gets a link (stretch, Bowei's app)

1. The creator opens a link from email or Twitter. No extension, no account.
2. The page is the video (YouTube embed) with Bowei's reactions on a timeline
   underneath, playing at their moments, exactly as in frame 5.
3. A reply box: "Reply to Bowei" (text or voice). Replies land in Bowei's app.

## Not designed yet (deliberately)

- Sign-in, invite links, who can see what. The demo hard-codes the circle.
- Playing reactions on a local copy (the audio-fingerprint stretch goal).
- Separate circles for wife, coworkers, creators. The demo has one list.
- Threads deeper than one reply.

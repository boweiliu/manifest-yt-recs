# manifest-yt-recs

Replace your YouTube recommendation feed with your circle.

Built for the "Dismantle Network Effects" build night, October 2026.

## The idea

YouTube decides what you watch next based on what keeps you watching, not on who you are. The people who know you would pick better, but they don't know precisely what you're into, and there's no good way for them to say "you'd love this, here's why."

So:

1. **Show your interests through reactions.** Record yourself reacting to a YouTube video in video, voice, text, or any combination, tied to the exact moment, including paused frames.
2. **Your circle sees them inside YouTube.** A browser extension you share with friends shows your reactions in their player at the moment you made them. It's like niconico, but richer than scrolling text. Nothing is posted publicly.
3. **They send you picks back.** Friends send videos from their own feeds, each with their reaction and why they think you'd like it. Those picks become your recommendation feed.
4. **Creators are in the loop.** By default, a video's creator can receive your reactions, so small creators get honest feedback and learn who their niche audience actually is.

The name comes from "manifestation": embody your interests in front of your circle, and they help you get there.

## Principles

- **Software for one.** You run the app and own the data: your reactions, and the picks your circle sends you. Nothing is stored on YouTube or another platform.
- **No accounts required.** People join through invite links sent over any channel. Google sign-in is optional, so friends who have left Google aren't excluded.
- **Separable from YouTube.** Reactions are tied to the video itself, not the YouTube page, so they can play on any copy of the same video.

## In this repo

The `docs/` folder is also the project's website (GitHub Pages, served from
`main` /docs, no build step):

| Page | Live | Source |
| --- | --- | --- |
| Landing page ("Handpicked.") | https://boweiliu.github.io/manifest-yt-recs/ | [`docs/index.html`](docs/index.html) |
| Mock screens, one page | https://boweiliu.github.io/manifest-yt-recs/storyboard/ | [`docs/storyboard/index.html`](docs/storyboard/index.html) |
| Mock screens, as slides | https://boweiliu.github.io/manifest-yt-recs/slides/ | [`docs/slides/index.html`](docs/slides/index.html) |

- [`docs/mocks/interactions.md`](docs/mocks/interactions.md): every interaction
  in the demo loop, written out.
- [`docs/api.md`](docs/api.md), [`docs/data-format.md`](docs/data-format.md):
  the server contract.

## Status

Spike for the build-night demo. The server contract is written; the server
itself isn't built yet.

- [`docs/api.md`](docs/api.md): the product flows as calls, and the endpoint reference.
- [`docs/data-format.md`](docs/data-format.md): the stored objects, and the metadata kept for debugging.

## License

MIT. See [LICENSE](LICENSE).

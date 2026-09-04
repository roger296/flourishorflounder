# Flourish or Flounder

The one-page website for **Flourish or Flounder** — Cate Hulme, Fractional CFO,
Board Advisor and Non-Executive Director.

Straight-talking financial advice for founders who want to actually understand
their numbers and what they should do next.

## What's here

A static site. No build step, no framework, no dependencies.

| File | Purpose |
|------|---------|
| `index.html` | The entire site — content, styling and behaviour in one file |
| `logo.png` | Full logo lockup, used as the hero headline |
| `mark.png` | The FOF mark, used small in the navigation bar |
| `favicon.png` / `favicon.ico` | Browser tab icons |
| `apple-touch-icon.png` | Icon for iOS home-screen shortcuts |
| `DEPLOY.md` | Deployment guide and editing instructions |

## Local preview

Open `index.html` in any browser. That's it — what you see is what deploys.

## Deployment

Hosted on a VPS via [Coolify](https://coolify.io) using the **Static** build
pack, publish directory `/`, port `80`. Pushing to `main` triggers a redeploy.

Full instructions, including DNS and SSL setup, are in [`DEPLOY.md`](DEPLOY.md).

## Editing

Search `index.html` for `EDIT ME` — the five placeholders needing real content
are numbered and documented inline. `DEPLOY.md` explains each one.

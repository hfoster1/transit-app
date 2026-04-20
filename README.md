# Doomben Station Live Trains

A lightweight, single-page web app that shows real-time arrivals and departures for **Doomben Station, Platform 1** (Brisbane, Queensland) using live TransLink data.

Live at: https://hfoster1.github.io/transit-app/

---

## How it works

```
GitHub Pages (static HTML)
        │
        │  fetch() every 30s
        ▼
Cloudflare Worker (CORS proxy)
        │
        │  TransLink GTFS Realtime API
        ▼
Queensland Rail live feed
```

The app is a single `index.html` file — no build step, no Node, no bundler. It uses React 18 and Tailwind CSS loaded from CDN. Because the TransLink API requires an API key and doesn't send CORS headers, a small [Cloudflare Worker](https://workers.cloudflare.com/) acts as a proxy that adds the key, fetches the data, and returns a clean JSON payload.

---

## Features

- **Next departure hero** — big countdown to your next train to the city
- **Full 90-minute schedule** — all upcoming arrivals and departures
- **Color-coded countdowns** — green (>10 min), yellow (5–10 min), red (<5 min)
- **Auto-refresh** — data refreshes every 30 seconds
- **Manual refresh** — tap the refresh button anytime
- **Loading skeletons** — no jarring blank states while data loads
- **Error handling** — clear message if the Worker or TransLink is unreachable

---

## Deployment

The app deploys automatically to GitHub Pages via GitHub Actions whenever you push to `main`.

### First-time setup

1. Go to **Settings → Pages** in your GitHub repo
2. Under *Source*, select **GitHub Actions**
3. Push any commit to `main` — the workflow in `.github/workflows/deploy.yml` will build and publish the site

The workflow uses the official `actions/deploy-pages` action, which is the current recommended approach for Pages deployments.

### Why Pages was failing

GitHub Pages requires either:
- A branch/folder source configured in repo settings, **or**
- A GitHub Actions workflow with `pages: write` and `id-token: write` permissions

Without the workflow file (`.github/workflows/deploy.yml`), there was nothing to trigger a deployment. Adding the workflow file and setting *Source → GitHub Actions* in repo settings fixes this.

---

## Cloudflare Worker

The Worker lives at:
```
https://proud-wood-b3ea.homehfoster.workers.dev/
```

It is responsible for:
- Authenticating with the TransLink API using a stored secret
- Filtering trips that stop at Doomben Station in the next 90 minutes
- Returning a simple JSON response:

```json
{
  "trains": [
    {
      "service": "Doomben Line",
      "scheduledTime": "14:32",
      "minutesUntil": "8 min"
    }
  ]
}
```

If you want to run your own instance, deploy a Worker with `TRANSLINK_API_KEY` set as an environment secret, then update the `WORKER_URL` constant in `index.html`.

---

## Local development

No build step needed — just open `index.html` in a browser:

```bash
# Option 1: open directly
open index.html

# Option 2: serve locally (avoids any browser fetch restrictions)
npx serve .
# or
python3 -m http.server 8080
```

The app will immediately try to hit the live Worker URL. If you're working on the Worker itself, update `WORKER_URL` to `http://localhost:8787` and run `wrangler dev` in the Worker project.

---

## Project structure

```
transit-app/
├── index.html                  # The entire frontend
├── README.md
└── .github/
    └── workflows/
        └── deploy.yml          # GitHub Pages deployment workflow
```

---

## Tech stack

| Layer | Technology |
|---|---|
| Frontend | React 18 (CDN), Tailwind CSS (CDN), Babel Standalone |
| Hosting | GitHub Pages |
| API proxy | Cloudflare Workers |
| Data source | TransLink GTFS Realtime |

---

## Limitations

- **Doomben-only**: The Worker is hardcoded to Doomben Station. Adapting it to another station requires updating the stop ID in the Worker.
- **No offline support**: There's no service worker or cached data — if the Worker is down, no trains are shown.
- **CDN dependency**: The app requires internet access even in local development to load React, ReactDOM, Babel, and Tailwind from their CDNs.

# Open AI Discovery Engine

A discovery engine for fantastic projects built on **open / open-weight AI models**. Each project is scored for **reuse, momentum, and rebuild difficulty**, and shown with live GitHub stars, interactive filters, charts, and a "how you'd rebuild it" sketch.

Deploys to **Cloudflare Pages** as a static front-end plus one cached API function.

---

## What's in here

```
oss-discovery-engine/
├── index.html                 # the app (self-contained; embeds a snapshot fallback)
├── functions/api/projects.js  # Cloudflare Pages Function — cached live-refresh API
├── data.json                  # the curated dataset (also embedded in index.html)
├── wrangler.toml              # Pages + KV config
└── README.md
```

## How it works

- **Open it offline** — double-click `index.html`. It runs entirely client-side on the embedded snapshot: search, filters, relevance slider, time window, charts, detail panels. The badge reads `snapshot`.
- **Deployed on Cloudflare** — the app calls `/api/projects` on load. The function returns a **server-cached** payload (KV, up to 6h old) to keep load and GitHub rate-limit usage low. The badge reads `live` and shows the last-refresh time.
- **Refresh data button** — calls `/api/projects?force=1`, which re-fetches current GitHub stats for every project, **recomputes the Radar scores**, caches the result with a fresh timestamp, and updates the view. The hand-written qualitative analysis (rebuild sketch, complexity, etc.) is preserved — only the numbers move.

## Deploy (public)

1. Install the CLI and log in:
   ```bash
   npm i -g wrangler
   wrangler login
   ```
2. Create the cache namespace and paste its id into `wrangler.toml`:
   ```bash
   wrangler kv namespace create RADAR_KV
   ```
3. (Recommended) add a GitHub token so refreshes don't hit the 60/hr anon limit:
   ```bash
   wrangler pages secret put GITHUB_TOKEN
   ```
4. Deploy:
   ```bash
   wrangler pages deploy .
   ```
   You'll get a public `*.pages.dev` URL. Add a custom domain in the Cloudflare dashboard if you like.

Local dev: `wrangler pages dev .`

## Make it private

Cloudflare **Pages → your project → Settings → Access policy** (or **Zero Trust → Access → Applications**): add an Access policy that allows only your email / Google / GitHub identity. The site then sits behind a login wall while staying on the same URL — no code changes.

## Merit vs. the static dashboard

| | Static HTML dashboard | This discovery engine |
|---|---|---|
| Opens offline | ✅ | ✅ (snapshot fallback) |
| Filters / search / charts | basic | focus, time window, relevance slider, free-text, scatter + bars |
| Live GitHub stars | frozen | refreshed on demand, server-cached |
| Re-score on fresh data | ❌ | ✅ |
| Shareable URL / access control | ❌ | ✅ public or private on Cloudflare |
| Cost | none | free tier covers it |

## Extending it

- **Grow the catalog:** add objects to `data.json` (and re-inject into `index.html` / `projects.js`, or just let the function read `data.json`). Keep `name` equal to the real `owner/repo` so live refresh can fetch it.
- **Auto-discover newcomers:** add GitHub *search* queries in `functions/api/projects.js` to surface trending repos not yet curated.
- **Scheduled refresh:** add a Cron Trigger that calls the function with `?force=1` so the cache is always warm.

> Scoring note: momentum is **proxy-based** (stars/day + recency + cross-source). For a truer read, add a tokened stargazer-timestamp pass to count stars gained inside a window.

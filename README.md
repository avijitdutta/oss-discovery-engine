# Open AI Discovery Engine

A discovery engine for fantastic projects built on **open / open-weight AI models**. Each project is scored for **reuse, momentum, and rebuild difficulty**, and shown with live GitHub stars, interactive filters, charts, and a "how you'd rebuild it" sketch.

Deploys as a **Cloudflare Worker with static assets** — a static front-end plus one cached API route, connected to GitHub so every push auto-deploys.

---

## What's in here

```
oss-discovery-engine/
├── public/
│   ├── index.html      # the app (self-contained; embeds a snapshot fallback)
│   └── data.json       # the curated dataset (also embedded in index.html)
├── src/index.js        # the Worker: serves /api/projects + falls back to static assets
├── wrangler.toml       # Worker + assets config (KV optional)
└── README.md
```

## How it works

- **Open it offline** — double-click `public/index.html`. It runs entirely client-side on the embedded snapshot: search, filters, relevance slider, time window, charts, detail panels. The badge reads `snapshot`.
- **Deployed on Cloudflare** — the app calls `/api/projects` on load. The Worker returns a **server-cached** payload (KV, up to 6h old) to keep load and GitHub rate-limit usage low. The badge reads `live` and shows the last-refresh time.
- **Refresh data button** — calls `/api/projects?force=1`, which re-fetches current GitHub stats for every project, **recomputes the Radar scores**, caches the result with a fresh timestamp, and updates the view. The hand-written qualitative analysis (rebuild sketch, complexity, etc.) is preserved — only the numbers move.

## Deploy (via GitHub → Cloudflare)

The repo is connected to a Cloudflare Worker with **Deploy command: `npx wrangler deploy`**. Just push:

```bash
git add -A && git commit -m "Workers + static assets" && git push
```

Cloudflare auto-builds on every push (or click **Retry build**). You'll get a `*.workers.dev` URL.

CLI alternative (no GitHub): `npm i -g wrangler && wrangler login && npx wrangler deploy`.
Local dev: `npx wrangler dev`.

## Enable caching (optional, after first deploy)

```bash
npx wrangler kv namespace create RADAR_KV
```

Uncomment the `[[kv_namespaces]]` block in `wrangler.toml`, paste the id, and push. (The KV id is an identifier, not a secret — safe to commit.)

## Higher GitHub rate limit (optional)

Anonymous GitHub allows 60 calls/hr; a token raises it to 5,000/hr:

```bash
npx wrangler secret put GITHUB_TOKEN
```

(Create a fine-grained token at github.com/settings/tokens — no scopes needed for public data.)

## Make it private

**Zero Trust → Access → Applications** → add a self-hosted app pointing at your Worker URL, with a policy allowing only your email. The site then sits behind a login wall, same URL, no code change.

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

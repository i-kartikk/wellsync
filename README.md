# WellSync

A single-file, installable web app (PWA) — food, water, exercise, step and
weight tracking with an Indian-first food database, streaks/XP/badges, a
rule-based coach, and WebGL water/calorie animations. No backend: all data
lives in the visitor's own browser (`localStorage`), so each install is
private to that device.

## Deploy to Vercel (~2 minutes)

**Option A — no install, drag and drop**
1. Go to https://vercel.com/new
2. Drag this whole folder onto the page (or "Browse" and select it).
3. Framework preset: choose **Other** / **Static**. Leave build command
   empty — there is nothing to build.
4. Click **Deploy**. You'll get a `https://your-project.vercel.app` URL.

**Option B — Vercel CLI**
```bash
npm i -g vercel      # one-time
cd wellsync-app
vercel                # first deploy, follow the prompts
vercel --prod          # promote to your production URL
```

**Option C — GitHub**
1. Push this folder to a new GitHub repo.
2. In Vercel: **New Project → Import** that repo.
3. Framework preset **Other**, no build command, output directory `.`.
4. Deploy. Every push to `main` redeploys automatically.

No environment variables, no database, no serverless functions — it's
static files only, so any static host works (Netlify, Cloudflare Pages,
GitHub Pages) if you'd rather not use Vercel. Just make sure `index.html`,
`manifest.webmanifest`, `sw.js` and the icon PNGs all sit at the **root**
of whatever URL you deploy to — the service worker registers at `/sw.js`
and the manifest at `/manifest.webmanifest`, both absolute paths.

## Installing it on your phone

Once it's live at a `https://` URL (installability requires HTTPS, which
Vercel gives you by default):

**Android (Chrome)**
- Open the URL. A card on the **Profile** tab reads "Install WellSync"
  with an **Install** button — tap it.
- Or use Chrome's menu → **Add to Home screen** / **Install app**.

**iPhone / iPad (Safari)**
- Safari doesn't support one-tap install prompts, so the same card shows
  a **How to** button with the three steps: Share icon → *Add to Home
  Screen* → *Add*.

Either way it then opens full-screen, no browser chrome, with its own
home-screen icon — a normal-looking app.

## Files

```
index.html              the whole app (HTML/CSS/JS, three.js pulled from cdnjs)
manifest.webmanifest     name, icons, theme colour, standalone display mode
sw.js                    service worker — caches the shell so it opens offline
icon-192.png             app icon
icon-512.png             app icon
icon-512-maskable.png    Android adaptive icon (safe-zone padded)
apple-touch-icon.png     iOS home-screen icon
vercel.json              static hosting config + correct headers for sw.js/manifest
```

## Updating it later

Edit `index.html` (or hand it back to Claude to edit) and redeploy — the
service worker uses a network-first strategy for the page shell, so
visitors get the new version on their next load without needing to
manually clear anything. If you change cached asset filenames, bump
`CACHE` in `sw.js` (e.g. `wellsync-v2`) so old caches get cleared.

## What this is not

There's no backend, no accounts, no server-side database — by design,
per the earlier build notes. Data is per-browser `localStorage`, so a
user's logs don't follow them from phone to laptop, and clearing site
data deletes them (there's an Export-to-JSON button in Settings for
backups). If you want real accounts and cross-device sync later, that
needs an actual backend (e.g. FastAPI + Postgres, as in the original
spec) behind this same frontend.

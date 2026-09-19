# WellSync

A single-file, installable web app (PWA) — food, water, exercise, step and
weight tracking with an Indian-first food database, streaks/XP/badges, a
rule-based coach, and WebGL water/calorie animations. It now supports account
sign-in with Firebase Authentication and cross-device synchronization through
Cloud Firestore. `localStorage` remains as an offline cache.

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

Firebase is the cloud backend. The frontend remains static, so it can still
be hosted on Vercel, Netlify, Cloudflare Pages or any HTTPS static host. Just make sure `index.html`,
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

## Multiple people, one install

The first time anyone opens the deployed link, they land straight in
onboarding — "what should we call you," their body stats, activity level
and goal. Finish it once and it's saved; refreshing or reopening the app
goes straight to the dashboard, not back through onboarding.

If a second person opens the *same* link on the *same* device (a shared
family phone, a kiosk tablet), WellSync shows a lightweight "who's this
for?" picker instead of overwriting the first person's data. Each name
gets its own fully separate set of logs, goals and streaks, stored under
its own key in `localStorage`. From Profile → Settings, anyone can
**Switch / Log Out** to go back to that picker (their data stays put) or
**Add / switch person** to set up someone new.

This is still per-device, not a real account system — two different
phones each get their own independent "who's this for" list, with no
sync between them. If several people install this on their *own* phones
from the same deployed link, they never see each other's data at all;
the picker only shows up when multiple people share one browser.

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

## Firebase setup — required before deployment

1. Create a Firebase project at https://console.firebase.google.com/
2. Add a **Web app** to the project.
3. Enable **Authentication → Sign-in method → Email/Password**.
4. Create a **Firestore Database**.
5. Open `index.html` and replace the six `PASTE_FIREBASE_*` values in
   `FIREBASE_CONFIG` with the config Firebase gives you.
6. In Firestore → Rules, publish:

```text
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{userId} {
      allow read, write: if request.auth != null
                         && request.auth.uid == userId;
    }
  }
}
```

7. Deploy the folder to Vercel.
8. Build the Android APK from the native Capacitor project using the same
   web app/backend configuration.

### Sync model

Firestore is the account source of truth. The browser/device keeps a local
cache so the app remains usable offline. Changes are uploaded after a short
debounce, and Firestore listeners update other signed-in devices in near real
time. Existing local data can be migrated into a newly created account.

**Important:** Firebase Web API keys are not passwords and are normally
included in client applications. Protect the data with Firestore Security
Rules; never put service-account/private keys in this frontend.

## What this is not

It is not a multi-user anonymous local-storage system anymore. A user's
cross-device identity is their Firebase account. Export-to-JSON remains
available as a backup.

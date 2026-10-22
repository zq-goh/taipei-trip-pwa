# Taipei Trip (offline PWA)

Files: index.html (the app), sw.js (offline caching), manifest.webmanifest (install info), icons/.
Your itinerary is stored on each device, not on the server. Save a backup file regularly (menu ⋯ > Back up or restore).

## Test on your laptop
    cd taipei-trip-pwa
    python3 -m http.server 8000
Open http://localhost:8000 in Chrome. DevTools > Application > Service workers should show sw.js activated.
DevTools > Network > set "Offline", reload: the app should still open.

## Deploy with GitHub Pages
1. Create a new repository (e.g. taipei-trip).
2. Add file > Upload files: upload index.html, sw.js, manifest.webmanifest and the icons folder. Commit.
3. Settings > Pages > Source: Deploy from a branch > main, / (root) > Save.
4. After a minute or two the app is at https://<your-username>.github.io/taipei-trip/

## Deploy without Git (Netlify Drop)
Drag the whole folder onto https://app.netlify.com/drop and use the URL it gives you.

## Install on your phone
- iPhone: open the URL in Safari > Share > Add to Home Screen.
- Android: open in Chrome > menu ⋯ > Install app (or the app's own "Install on this device" button).
Open it once while online, then test in airplane mode.

## Updating the app
Change the files, then bump CACHE_VERSION in sw.js (e.g. taipei-trip-v2) and re-upload.
Open the app online; it will show "A new version is ready" > Reload. Your trip data is kept.

## Moving your trip from the Claude version
In the Claude version: ⋯ > Back up or restore > Copy.
In this app: ⋯ > Back up or restore > paste over the text > tap "Replace trip with this text" twice.

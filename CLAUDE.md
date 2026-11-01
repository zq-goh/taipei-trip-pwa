# Taipei Trip PWA

Personal itinerary app for a Taipei trip (November 2026). Used on a phone during the trip, often with poor or no connection.

## Structure
- `index.html`: the whole app (HTML, CSS, JS inline). No build step, no framework, no dependencies.
- `sw.js`: service worker. Precaches app files (cache-first) so the app opens offline.
- `manifest.webmanifest`, `icons/`: installability (Add to Home Screen).
- `trip.enc.json` (optional, not in `APP_FILES`): the encrypted shared trip for family view. Created by the owner in
  the app (⋯ > Share with family) and uploaded to the repo. The app fetches it network-first; it is never precached.
- Data: one JSON object in `localStorage` under key `taipei-trip:v1`; UI state under `taipei-trip:ui`;
  family view's decrypted copy and passphrase under `taipei-trip:view` (that device only).
  Shape: `{version, name, docUrl, docMsg, docPic, updatedAt, days:[{id, title, date:"YYYY-MM-DD", stops:[{id, time:"HH:MM", name, zh, en, address, category, notes, done}]}]}`

## Design decisions (keep unless asked)
- Each day takes a warm colour (vermilion, amber, plum, persimmon, oolong tea, rose; all white-text readable, owner's
  request, Nov 2026). Day codes are formal (banking-style) Chinese numerals (壹, 貳, 參...) shown in a 斗方 (gold-edged
  diamond of New Year door paper) on the day tabs, day header and stamp book; stops are numbered 01, 02, 03... within
  the day, with no prefix. The 第一天 kicker keeps everyday numerals.
- Light-hearted Taiwan theme: warm palette, rounded system font, done stops get a 已到 souvenir stamp (紀念章),
  short Traditional Chinese touches (第一天, 下一站, 終點站, trip countdown) next to English, inline SVG Taipei 101 skyline.
  Chinese is always paired with English so the app stays readable.
- Illustrations are hand-drawn inline SVG in `<template id="art-*">`, cloned with `art(name)`: day banner (temple roof
  with swallowtail ridges, red lanterns, Taipei 101, sky lanterns), black bear with bubble tea (empty day),
  rising sky lanterns with 平安 and plum blossoms (all done). Decorative only (aria-hidden).
- "Navigate" appears only when a stop has an address (needs connection; that's accepted). `address` holds either a
  plain address (opens a Google Maps search) or a Google Maps link (opened directly). `mapsLink()` accepts only
  Google Maps hosts (maps.app.goo.gl, goo.gl/maps, google.<tld>/maps, maps.google.<tld>) and rebuilds the URL as
  https; anything else is treated as search text, so restored data can't produce a non-Google or javascript: href.
  The editor keeps just the link from pasted share text and blocks saving other sites' links.
- Stops may carry `en`: an optional small English line under the place name, so a trip written in Chinese (the
  owner's real one is in Simplified Chinese for senior family members, Nov 2026) still has English. Old backups
  without it restore fine.
- Header 📄 icon button ("原版行程 Original plan", next to the theme toggle) opens a popup with the owner's thank-you
  message (`trip.docMsg`, plain text, line breaks kept), an optional picture (`trip.docPic`) and an "Open link" button
  to the original itinerary PDF on Google Drive (`trip.docUrl`). All three are set in Trip settings and live in the
  trip, so they reach family only inside the encrypted `trip.enc.json` and work offline (except opening the link);
  never commit them in readable form. The button shows when any of the three is set. `driveLink()` accepts only
  drive.google.com / docs.google.com and rebuilds the URL as https. The picture is shrunk in the app (max 900px JPEG,
  ~300 KB) and `picOk()` accepts only jpeg/png/webp base64 data URLs up to 1 MB, so restored data can't load anything else.
- No "Show driver" screen: removed at the owner's request (Oct 2026). The Chinese name still shows on each card.
- System fonts only, so nothing loads from the network.
- Destructive actions use two-tap confirm buttons, not `confirm()`.
- Stamp book (集章冊) dialog: every stop as a stamp, grouped by day; tapping one jumps to that stop. Opened from
  "📖 Stamp book" by Edit day and from the all-done link. Read-only view of `done`; marking done stays on the badge.
- Swipe left/right on `main` (or arrow keys) changes day with a short slide. Swipes from within 24px of the screen
  edge are ignored so the browser's back gesture still works.
- Family view (view-only): `?view` link, remembered per device until "Leave view-only mode". Lock screen asks for the
  passphrase, then shows the trip with no editing (no add/edit/delete/stamping/settings; stamp book, swipe, Navigate,
  theme still work). On each online open it re-fetches `trip.enc.json` and updates; a changed passphrase relocks it.
  Never touches the owner's `taipei-trip:v1`. Encryption: PBKDF2-SHA256 (600k iterations) -> AES-256-GCM, WebCrypto only.
- Light theme by default, regardless of the phone's setting (owner's request, Nov 2026). The header sun/moon button
  switches to dark; `ui.theme` ("light"/"dark", in `taipei-trip:ui`, not in trip backups) remembers it, and only
  "dark" sets `data-theme="dark"` on `<html>`. A tiny `<head>` script applies it before first paint.

## Rules
- Every change to any cached file: bump `CACHE_VERSION` in `sw.js` and update `APP_FILES` if files are added or removed.
- No external scripts, fonts, or CDNs. Everything must work in airplane mode after first load.
- Use relative paths (`./`) so it works on GitHub Pages project sites.
- Keep the backup format backward compatible (old backup files must still restore).
- Never commit the owner's real itinerary, bookings, or personal details in readable form. The only committed copy is
  `trip.enc.json`, encrypted in the app with the owner's passphrase (owner's decision, Oct 2026). Never ask for,
  store, log, or commit the passphrase, and never commit plaintext backups (`taipei-trip-backup-*.json` is ignored).
  Test encrypted files are made from the sample trip in a scratch copy, never in the repo folder.
- Build user-visible text with `textContent` / the `h()` helper, never `innerHTML` with user data.

## Testing
- Serve: `python3 -m http.server 8000`, open http://localhost:8000.
- Offline check: load once, go offline (DevTools or Playwright `context.set_offline(True)`), reload; stops and done states must persist.
- Check light and dark mode and a ~380px-wide viewport.

## Deployment
GitHub Pages from `main`, root folder. URL: https://<username>.github.io/<repo>/

## Backlog (owner's priorities)
1. Load the real itinerary (dates, places).
2. Rainy-day alternatives per day.
3. Food / night-market wishlist with tried + rating.
4. Expense log in TWD with MYR conversion (manual rate; no live API needed offline).

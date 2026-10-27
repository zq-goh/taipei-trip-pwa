# Taipei Trip PWA

Personal itinerary app for a Taipei trip (November 2026). Used on a phone during the trip, often with poor or no connection.

## Structure
- `index.html`: the whole app (HTML, CSS, JS inline). No build step, no framework, no dependencies.
- `sw.js`: service worker. Precaches app files (cache-first) so the app opens offline.
- `manifest.webmanifest`, `icons/`: installability (Add to Home Screen).
- Data: one JSON object in `localStorage` under key `taipei-trip:v1`; UI state under `taipei-trip:ui`.
  Shape: `{version, name, updatedAt, days:[{id, title, date:"YYYY-MM-DD", stops:[{id, time:"HH:MM", name, zh, address, category, notes, done}]}]}`

## Design decisions (keep unless asked)
- Days take Taipei Metro line colours/letters (R, BL, G, O, BR, Y); stops show station-style codes (R01, R02...).
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
- No "Show driver" screen: removed at the owner's request (Oct 2026). The Chinese name still shows on each card.
- System fonts only, so nothing loads from the network.
- Destructive actions use two-tap confirm buttons, not `confirm()`.
- Stamp book (集章冊) dialog: every stop as a stamp, grouped by day; tapping one jumps to that stop. Opened from
  "📖 Stamp book" by Edit day and from the all-done link. Read-only view of `done`; marking done stays on the badge.
- Swipe left/right on `main` (or arrow keys) changes day with a short slide. Swipes from within 24px of the screen
  edge are ignored so the browser's back gesture still works.
- Header sun/moon button toggles light/dark. It follows the phone's setting until first used, then `ui.theme`
  ("light"/"dark", in `taipei-trip:ui`, not in trip backups) sets `data-theme` on `<html>`. A tiny `<head>` script
  applies it before first paint.

## Rules
- Every change to any cached file: bump `CACHE_VERSION` in `sw.js` and update `APP_FILES` if files are added or removed.
- No external scripts, fonts, or CDNs. Everything must work in airplane mode after first load.
- Use relative paths (`./`) so it works on GitHub Pages project sites.
- Keep the backup format backward compatible (old backup files must still restore).
- Never commit the owner's real itinerary, bookings, or personal details to the repo; the trip lives only in localStorage.
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

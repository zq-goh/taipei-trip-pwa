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
- "Show driver" shows the Chinese name and address full screen with 請帶我到這裡.
- "Navigate" opens a Google Maps search URL (needs connection; that's accepted).
- System fonts only, so nothing loads from the network.
- Destructive actions use two-tap confirm buttons, not `confirm()`.

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

# Vacation Optimizer

A single-page, self-contained prototype that helps a user find the **cheapest time to travel** when combining flight + hotel + cruise.

## What it does

- User enters: destination (or a preset), earliest/latest departure, trip length in nights, adults, children, home airport (optional), max budget (optional), and toggles for flight, hotel, and cruise.
- The app compares **every departure date** in the window, builds a total trip cost per date, and ranks windows cheapest first, with:
  - total cost big, breakdown small: flight / hotel / cruise / taxes / bundle discount
  - per-person cost, savings vs. the most expensive window
  - a bar chart of total cost by departure date (green = cheapest, red = most expensive)
  - a **price calendar**: a month-grid heatmap (green = bottom quartile, yellow = typical, red = top quartile), clickable to select a date
- Prices are **estimates, not live prices**, until real APIs are connected (see below).

## New in v2

- **Price calendar** (month grid, clickable) — kept alongside the existing bar chart.
- **Flexible dates:** a ±3 days toggle shows, for each candidate, the cheapest total within 3 days of it. A **Cheapest month** summary (average total by departure month) appears in the sticky results summary.
- **Filters and sorting:** sort by cheapest total, cheapest per person, or best value. Best value is defined clearly in the app: total ÷ (1 + 0.08 × weekend nights in the trip) — trips covering more weekend nights score better, because those are days you do not need to take off work. Filters: max budget, hotel stars (3★ 0.85×, 4★ 1.00×, 5★ 1.35× nightly), cruise length (Short ≤4 nights, Medium 5–6, Long 7+), hide weekend departures (Sat/Sun), only direct flights (+8% flight, estimate).
- **Bundle vs separate:** total includes a **bundle discount line** — 10% off hotel when flight + hotel are both checked. This is an estimate, **not a real Expedia offer**. The table shows flight + hotel + cruise + taxes − bundle discount = total, so you can see the separate components and the bundled total.
- **Compare:** pin up to 3 departure windows (Pin button per row, click again to unpin) and see them side-by-side in a comparison table: dates, total, per person, flight, hotel, cruise, taxes, bundle discount, savings.
- **Book now or wait indicator:** simple, clearly labeled estimate — bottom quartile of totals in your window = “Low vs typical”, middle = “Typical”, top quartile = “High vs typical”. This is a comparison inside your own window, **NOT a real price prediction**.
- **Price watch:** “Save this search” stores the search in `localStorage` (browser only, no server, no account). Saved searches show on load, with Re-run and Delete buttons.
- **Destination presets:** Caribbean, Mexico, Alaska, Europe, Hawaii, Florida — each uses different flight / hotel / cruise base prices, so presets actually change results. A custom destination falls back to Caribbean base prices plus its own deterministic variation.
- **Better UI:** clean top search card, sticky results summary (cheapest total, cheapest month, book indicator), mobile-responsive layout, larger touch targets (48px+ inputs/buttons), total-cost-first hierarchy, and an **empty state** when no dates match filters, with a Clear filters button.
- **Fees/taxes line item (estimate):** 12% on hotel, 8% on cruise, labeled as an estimate. No flight tax in this model.

## How to use compare and price watch

- **Compare:** run a search, click **Pin** on up to 3 rows. A comparison table appears below the calendar. Click Pin again on a pinned row to unpin it. Trying to pin a 4th window shows a “Compare is full” message and does not add it.
- **Price watch:** after filling in a trip, click **Save this search (price watch)**. Saved searches appear under the form on your next visit (same browser). Click **Re-run** to load that search and run it again, or **Delete** to remove it. Nothing is sent to a server.

## Installable iPhone app (PWA) — no App Store needed

This is a Progressive Web App. It can be added to an iPhone Home Screen and opened full-screen, like a native app, with no developer account and no App Store review.

### Install on iPhone
1. Host the app on an **https://** site first (GitHub Pages below is the simplest — the raw `index.html` file opened locally will **not** install, and Safari will not offer Add to Home Screen for `file://` pages).
2. On your iPhone, open the hosted URL in **Safari** (not an in-app browser).
3. Tap **Share** (square with an arrow pointing up) → scroll down → **Add to Home Screen** → **Add**.
4. Open it from the new **Vacation** icon on your Home Screen. The in-app install card repeats these steps, and shows a "running the installed app" note when opened standalone.

### Host it on GitHub Pages (repo owner must enable this)
GitHub Pages cannot be enabled from the app code or from an agent — the repository owner enables it:
1. In the GitHub repo, go to **Settings → Pages**.
2. Under **Build and deployment**, choose **Deploy from a branch**, branch **main**, folder **/ (root)**.
3. Save. GitHub will publish the site at an `https://<owner>.github.io/<repo>/` URL (it can take a minute or two on first deploy).
4. Open that URL in Safari on the iPhone and follow the install steps above.

### What the PWA includes
- `manifest.webmanifest` — name "Vacation Optimizer", short_name "Vacation", `display: standalone`, theme/background `#0f172a`, `start_url`/`scope` `./`, icons 192 and 512 (plus a 512 maskable variant).
- `sw.js` — service worker caching the app shell (`index.html`, manifest, icons) for offline use, with a versioned cache (`vacation-optimizer-v1`) that purges old caches on activate. It only handles same-origin GET requests and never caches API responses or keys.
- iOS head tags — `apple-mobile-web-app-capable`, status bar style `black-translucent`, title "Vacation", `apple-touch-icon` (180×180), `viewport-fit=cover`, `theme-color`.
- Registration in `index.html` is guarded: it only runs over `http:`/`https:` and only if service workers are supported, so opening the file locally still works (just without offline caching or installability).
- Mobile-first polish — safe-area insets (`env(safe-area-inset-*)`) on header/main, a sticky bottom-anchored search button on small screens, inputs at 16px+ so iOS does not zoom on focus, single-column layout under 850px, and results/compare tables in horizontally scrollable wrappers instead of being clipped.

### Offline behavior
Once installed (or visited once over https), the app shell loads offline. Saved searches (localStorage) and all v2 features still work offline, because pricing is computed locally. Live API calls, if wired in later, would still need a connection — the service worker deliberately does not cache them.

## How to run it

No build step, no server needed:

1. Open `index.html` in any modern browser (double-click it, or drag it into a browser window).
2. Fill in the trip form. The defaults are a useful sample: Caribbean, Jan 1–31, 2027, 7 nights, 2 adults, flight + hotel + cruise on.
3. Press **Find cheapest windows**.

That's it — vanilla HTML/CSS/JS only, no external libraries. Note: running it this way is fine for testing, but it will **not** be installable — for Add to Home Screen, host it over https (GitHub Pages above).

## How the estimate works

- **Flight:** preset base round-trip per adult, adjusted by weekday (Tue/Wed cheapest, Fri/Sun dearest), season by month, plus a deterministic pseudo-random variation per date + destination. Children at 75% of adult flight price. Direct-only adds 8%.
- **Hotel:** preset nightly rate × star multiplier, with a Fri/Sat-night weekend premium, season, and per-date variation.
- **Cruise:** preset base per person by length category — Short ≤4 nights (50% of long base), Medium 5–6 nights (72%), Long 7+ nights (100%) — children at 50%.
- **Taxes/fees (estimate):** 12% on hotel, 8% on cruise.
- **Bundle discount (estimate, not a real offer):** 10% off hotel when flight + hotel are both checked.
- **Total = flight + hotel + cruise + taxes − bundle discount.** **Per-person** = total ÷ (adults + children).

## Connecting live APIs

The page has a clearly marked **"Live prices"** section with:

- API key fields for Duffel (flights), a hotel provider, and a cruise provider — **entered in the browser only, never hardcoded, never saved to git**. Fields are session-only.
- Placeholder functions in the page source:
  - `fetchLiveFlight()` → Duffel Offers API (`POST https://api.duffel.com/air/offers`)
  - `fetchLiveHotel()` → a hotel booking API for nightly rates by date
  - `fetchLiveCruise()` → a cruise line / aggregator API for sailing price by departure date

Each placeholder currently returns `null`, which tells the app to fall back to estimates. To go live:

1. Wire each function to return a per-date price in the same shape as the estimate functions (or `null` on failure).
2. In production, do **not** ship API keys in client-side code — use a server proxy / environment variables, and keep keys out of the repository.

## Changelog

### PWA (2026-11-04) — v2 → installable iPhone app
`manifest.webmanifest`, `sw.js` app-shell offline cache, iOS head tags + 180/192/512 (+512 maskable) PNG icons generated locally with PIL, in-app "Install on iPhone" card (Share → Add to Home Screen, https required), safe-area insets, sticky search button on small screens, 16px+ inputs, scrollable results/compare tables. All v2 features unchanged: price calendar, ±3 days, filters/sorting, compare up to 3, bundle pricing, presets, price watches, live-API placeholders. Not pushed to GitHub — pushing is left to the parent agent. GitHub Pages must be enabled by the repo owner (Settings → Pages → Deploy from branch main, root).

### v2 (2026-11-04)
Price calendar heatmap, ±3-day flexible dates, cheapest-month summary, sort (total / per person / best value), filters (budget, stars, cruise length, hide weekends, direct-only), bundle discount line, compare up to 3, book-now-or-wait quartile indicator, price watch via localStorage, destination presets, sticky summary, mobile touch targets, taxes/fees line, empty state. Total formula changed to flight + hotel + cruise + taxes − bundle discount.

### v1
Initial prototype: destination/window/nights/party inputs, flight+hotel+cruise ranking, bar chart, live-API placeholders, estimates only.

## GitHub status

This prototype was built locally in `~/workspace/vacation-optimizer/` and pushed to GitHub in v1 (see the parent agent's delivery for the repo/commit). **This v2 refinement was NOT pushed to GitHub** — per the refinement task, pushing is left to the parent agent.

## Test sample — v2

Sample run (verified with Node, extracting the script from index.html): Caribbean, Jan 1–31, 2027, 7 nights, 2 adults, flight + hotel + cruise.

- 31 departure dates evaluated, ranked ascending by total: **true**
- Every row: flight + hotel + cruise + taxes − bundle discount = total (within $0.01): **true**, no NaNs: **true**
- Cheapest window: **2027-01-27 → 2027-02-03, $3,603 total / $1,801 per person** (flight $676, hotel $1,081, cruise $1,690, taxes $265, bundle discount −$108, saving $688 vs. max)
- Cheapest month: 2027-01 (average $3,914, n=31)
- Calendar cells match the ranked list totals: **true**
- Filters actually remove results: hide weekends 31 → 21, budget $3,000 → 0 (empty state), cruise Short with 7 nights → 0 (empty state)
- Compare pins capped at 3: **true**
- Flexible ±3 example: 2027-01-21 ($3,843) → flex $3,659 on 2027-01-18
- Presets change results: Mexico cheapest $3,066, Europe cheapest $4,185, vs. Caribbean $3,603
- Sort by per person and by best value both ascending: **true**
- Direct-only changes flight price: **true**

No test failures in the Node checks above. Browser-only features (calendar clicks, pin UI, localStorage price watch) were not exercised in Node; they are guarded for browser use.

## Testing the ranking math

```bash
node -e "
const fs = require('fs');
const src = fs.readFileSync('index.html','utf8').match(/<script>([\s\S]*?)<\/script>/)[1];
const m = {exports:{}}; const fn = new Function('module','exports','document', src); fn(m, m.exports, undefined);
const {sorted} = m.exports.generateResults({destination:'Caribbean',earliest:'2027-01-01',latest:'2027-01-31',nights:7,adults:2,children:0,useFlight:true,useHotel:true,useCruise:true,budget:null});
console.log(sorted.slice(0,3));
console.log('sorted?', sorted.every((r,i)=> i===0 || sorted[i-1].total <= r.total));
console.log('sums ok?', sorted.every(r=> Math.abs(r.flight+r.hotel+r.cruise+r.taxes-r.bundleDiscount-r.total) < 0.01));
console.log('no NaN?', sorted.every(r=> Number.isFinite(r.total)));
"
```

## Limitations

- Estimates only until APIs are connected — do not book on these numbers.
- Cruise + hotel are summed as selected; a real cruise already includes a cabin, so a future version may offer a "cruise replaces hotel nights" mode.
- No baggage, transfers, or cabin-category modelling yet.
- Price watch is local to one browser (localStorage); it does not notify you of price changes.

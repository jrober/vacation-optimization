# Vacation Optimizer

A single-page, self-contained prototype that helps a user find the **cheapest time to travel** when combining flight + hotel + cruise.

## New in v4 — Exact bundles are the main flow (not pushed)

Exact named airline + named hotel bundles are now the **first section in the app**, before the estimate engine. The estimate engine is moved below it and labeled "Estimate only — use when your destination has no exact data yet", and estimate totals are never mixed into an exact bundle. The price calendar is also labeled estimate-only.

- **Exact destination selector:** Cancun, Orlando, Honolulu. Switching it swaps the exact flight list, hotel list, bundles, best-value pick, and route/source lines. All three are SLC origin, 2 adults, economy, depart 2027-01-27, return 2027-02-03, 7 nights.
  - **Cancun (searched 2026-11-04):** Duffel, coverage complete, 2,339 offers; Expedia, 50 hotels. Cheapest Flight + Hotel: **Southwest Airlines $1,096.74** (08:40 → 19:45 out, WN3980/WN1450/WN0307; 17:20 → 11:45+1 back, WN0306/WN1852/WN0971; 4 stops total) + **Hotel Hacienda de Castilla $270.47** (3.0★, 7.4, Standard Room 2 Double Beds) = **$1,367.21 total, $683.61 per person**.
  - **Orlando (searched 2026-11-05):** Duffel, coverage complete, 2,445 offers; Expedia, 50 hotels. Cheapest Flight + Hotel: **Frontier Airlines $539.96** (07:40 → 16:21 out, F93862/F93312; 19:20 → 00:04+1 back, F92519/F94251; 2 stops total) + **Rodeway Inn Kissimmee Maingate West $413.98** (2.0★, 6.8, Room 1 Double Bed) = **$953.94 total, $476.97 per person**.
  - **Honolulu (searched 2026-11-05):** Duffel, coverage complete, 1,606 offers; Expedia, 50 hotels. Cheapest Flight + Hotel (private room): **Southwest Airlines $1,010.22** (06:50 → 18:55 out, WN0949/WN3373/WN2022; 23:20 → 11:10+1 back, WN1773/WN3612; 3 stops total) + **Ewa Hotel Waikiki $976.52** (2.0★, 7.6, Room 1-Moderate) = **$1,986.74 total, $993.37 per person**. Note: the cheapest Honolulu hotel overall is Bposhtels Waikiki Retreat at $684.95, but it is a shared dormitory (women only), not a private room for 2, so it is listed last in the app and is not the default bundle hotel. With it, the total would be $1,695.17 — do not use that as a 2-adult private-room bundle.
- **Every bundle names both sides:** each bundle row shows the selected airline + flight numbers + times + stops and the selected hotel + stars + guest review + room, plus total and per-person total. No bundle shows a total without naming its flight and hotel. Pick a different flight (5 options per destination) or hotel (5 options per destination) with the radio buttons and every bundle total recalculates by exactly the price difference.
- **Best value (new definition):** cheapest flight + the highest guest-review hotel in that destination's exact list. Cancun: Southwest $1,096.74 + Hotel Los Girasoles Cancun (8.8, $339.29) = $1,436.03. Orlando: Frontier $539.96 + Ramada by Wyndham Kissimmee Gateway (8.2, $535.04) = $1,075.00. Honolulu: Southwest $1,010.22 + Ramada Plaza by Wyndham Waikiki (8.6, $1,052.49) = $2,062.71.
- **Cruise and theme parks stay gated:** cruise and theme-park checkboxes now also sit inside the exact section (synced with the trip form below), and both are still completely hidden / counted as $0 when unchecked. Cruise is an estimate, theme parks are estimates — neither is presented as exact.
- **PWA:** service-worker cache bumped to `vacation-optimizer-v4`.
- **Snapshot, not live:** all exact prices are from the searches above and will change. Re-check before booking. Nothing is booked and no payment is taken.

### How to add a new exact destination
1. Run a Duffel search for the new route on the same pattern (origin, destination, depart/return dates, 2 adults, economy, `--limit 300`) and an Expedia guest search for the destination city on the same check-in/check-out dates (`--room 2 --sort-type totalpricewithfees`). Save the raw outputs under `data/` — do **not** push `data/` (raw outputs are ~1–2 MB each and Expedia result URLs contain session parameters).
2. Pick at least 3 flights (cheapest per airline is a good default) and at least 3 hotels (cheapest private rooms first; list any shared-dormitory result last with a clear note, never as the default bundle hotel). Copy the exact airline, flight numbers, times, stops total, duration, total; and hotel name, stars, review score/count, room, total including taxes and fees, refundable flag.
3. Add a new entry to `EXACT_DATA` in `index.html` in the same shape as `cancun`/`orlando`/`honolulu` (key, label, route, origin, destination, departure, returnDate, nights, adults, searchedOn, flightSource, hotelSource, cruiseEstimate, cruiseTaxRate, flights[], hotels[]), add a destination button with `data-exact-dest="<key>"` in the exact section, and bump the service-worker cache in `sw.js`. The picker, bundles, best-value pick, and per-person math then work without further code changes.
4. For live any-destination search (not a snapshot), build the backend step in "Architecture" below instead of hand-adding datasets.

## New in v3 — Exact options and bundles (demo dataset)

- **Exact options mode:** a new "Exact options — SLC → Cancun demo" section in `index.html`, for one real searched trip: Salt Lake City (SLC) → Cancun (CUN), depart 2027-01-27, return 2027-02-03, 7 nights, 2 adults, economy. It shows real flight options and real hotel options side by side, each with exact airline / times / stops / room / total — not a single blended estimate.
  - Flights (Duffel, searched 2026-11-04, coverage complete, 2,339 offers scanned): Southwest $1,096.74 (08:40 → 19:45 out, 17:20 → 11:45+1 back, 4 stops total), Southwest $1,096.74 (07:30 outbound variant), Aeromexico $1,380.54, Delta $1,389.54 (1 stop total), Delta nonstop $2,288.54.
  - Hotels (Expedia guest search, searched 2026-11-04, 50 hotels returned, sorted by total including mandatory taxes and fees; hotel results via Expedia): Hotel Hacienda de Castilla $270.47 (3.0★, review 7.4), Hotel Colonial Cancun $292.07 (3.0★, 7.6), Suites Cancun Center Cancun $296.37 (2.5★, 7.4), Terracaribe $311.84 (3.0★, 7.8), Hotel Los Girasoles Cancun $339.29 (3.0★, 8.8).
- **Bundles:** Flight only, Hotel only, Flight + Hotel, Flight + Hotel + Cruise (cruise estimate, labeled), and Best value. Each bundle shows its breakdown and per-person total. Pick a different flight or hotel with the radio buttons and every bundle total recalculates. Pin up to 3 bundles to compare them side by side.
  - Cheapest Flight + Hotel bundle: $1,096.74 + $270.47 = **$1,367.21 total, $683.61 per person**.
  - Cheapest Flight + Hotel + Cruise bundle: $1,096.74 + $270.47 + $1,689.60 cruise estimate + $135.17 cruise tax = **$3,191.98 total, $1,595.99 per person**.
  - Best value bundle (definition fixed in the app): cheapest flight + the highest guest-review hotel with a total of $350 or less in this Expedia result set = Southwest $1,096.74 + Hotel Los Girasoles Cancun $339.29 = **$1,436.03 total**.
  - Bundle math rule: hotel totals from Expedia already include mandatory taxes and fees, and flight totals are the Duffel totals, so no extra tax is added to those lines. The old v2 10% bundle discount is **not** applied to exact offers — it was an estimate, not a real offer.
- **Cruise stays an estimate / link-out:** there is no cruise connector and no usable public cruise-price API. Expedia Cruises is a website search, and commercial cruise APIs (Widgety, Cruise Itinerary, Travelopro) need a key or contract. No cruise price is presented as exact in v3. The app says to search Expedia Cruises or a cruise line site directly for the same dates.
- **Snapshot, not live:** the demo flight/hotel prices are from the 2026-11-04 searches above and will change. Re-check before booking. Nothing in the app is booked and no payment is taken.
- PWA intact: manifest, service worker (cache bumped to `vacation-optimizer-v3`), iOS tags, mobile layout. All v2 features kept: price calendar, filters/sorting, compare up to 3 windows, destination presets, price watch.

## Architecture — why the live app cannot yet show live prices, and what is needed

A static page on GitHub Pages cannot call Duffel or Expedia directly in the browser: those APIs need a server-side secret key, and a key shipped in public page code would be exposed to anyone who loads the page, and browser CORS rules block direct calls from a static origin. That is why v3's in-app data model is structured for exact offers (`EXACT_DATA` flights/hotels + `buildBundle()` in `index.html`), with a real demo dataset embedded so the bundle UI is demonstrable today — but the demo data is **not** live data.

The documented backend step still needed before the live app shows live prices:
1. Run a small server (or serverless function) that holds the Duffel and Expedia keys in environment variables — never in the repo or page code.
2. That server exposes one endpoint, e.g. `GET /api/offers?origin=SLC&destination=CUN&depart=2027-01-27&return=2027-02-03&adults=2`, which calls Duffel Offers and the Expedia lodging search, normalises each offer to the `EXACT_DATA` shape (airline, outbound/return times, stops, total; hotel name, stars, review score, room, total including taxes and fees), and returns JSON.
3. The app then fetches that endpoint instead of using `EXACT_DATA`, keeping the same bundle math and UI. Cruise needs a separate commercial cruise API key/contract added to that same server before it can be exact; until then it stays an estimate with a link-out.
4. Cache the server response briefly (minutes, not days) and show the search timestamp in the app, because flight/hotel prices change and a stale exact price is worse than a labeled estimate.

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
- `sw.js` — service worker caching the app shell (`index.html`, manifest, icons) for offline use, with a versioned cache (`vacation-optimizer-v3` as of v3) that purges old caches on activate. It only handles same-origin GET requests and never caches API responses or keys.
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

### v4 (2026-11-05) — Exact bundles first (not pushed)
Exact bundles moved to the top of the app, estimate engine moved below and labeled estimate-only, exact destination selector (Cancun / Orlando / Honolulu) with 5 flights and 5 hotels each, every bundle row naming its airline and hotel, best value redefined as cheapest flight + highest guest-review hotel in the exact list, cruise/theme-park toggles added inside the exact section (synced, still gated), service-worker cache bumped to v4. New live searches only (no bookings): Duffel SLC → MCO and SLC → HNL, Expedia Orlando and Honolulu, all Jan 27–Feb 3, 2027, 2 adults. Honolulu default bundle uses the cheapest private room (Ewa Hotel), not the cheaper shared dormitory. Not pushed to GitHub — pushing is left to the parent agent. Do NOT push `data/`.

### v3.1 (2026-11-04) — Cruise gating fix + theme parks (not pushed)
- **Cruise gating fix:** cruise is now unchecked by default and completely hidden when unchecked — no cruise column, cruise bundle, cruise note, or cruise amount in any estimate total, breakdown, compare table, calendar total, or exact bundle. Checking cruise restores it exactly as before, still labeled as an estimate (no cruise connector). The same rule applies to every component: it only contributes to a total when its checkbox is on.
- **Theme parks (estimate only):** new “Theme parks” checkbox with Walt Disney World (Orlando, $165 1-day adult), Disneyland (California, $160), Universal Orlando ($140), Universal Hollywood ($130), Legoland Florida ($110), SeaWorld Orlando ($105); children at 90% of adult. Park days 1–7 (default = trip length, capped at 7); multi-day per person = 1-day price × days^0.75, so average price per day falls as days rise. No theme-park ticket API connector exists, so prices are labeled estimates — check the park's official site for exact ticket prices. New bundles when checked: Flight + Hotel + Theme Parks, and Flight + Hotel + Theme Parks + Cruise (only when both are checked), pinnable in compare like other bundles. Orlando/Anaheim/Florida presets show a theme-park suggestion without forcing it on; Caribbean/Mexico/Alaska/Europe presets do not suggest parks.
- v3 exact flights/hotels, PWA, price calendar, filters, and compare-up-to-3 are unchanged.

### v3 (2026-11-04) — Exact options and bundles
Exact options section with a real SLC → Cancun demo dataset (Duffel flights, Expedia hotels, searched 2026-11-04), five bundles with recalculating totals and per-person totals, pin-up-to-3 bundle comparison, Best value definition (cheapest flight + highest-review hotel ≤ $350), cruise kept as a labeled estimate with no exact price invented, service-worker cache bumped to v3, and an architecture note documenting the backend/proxy step still needed for live prices. Nothing booked, no payment wallet used. **Not pushed to GitHub — pushing is left to the parent agent. Do NOT push `data/` (raw search outputs, ~2.8 MB, and Expedia result URLs contain session parameters).**


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
- v4 exact data covers only SLC → Cancun (2026-11-04), Orlando (2026-11-05), and Honolulu (2026-11-05), Jan 27–Feb 3, 2027, 2 adults, as snapshots. Other routes/dates still use the estimate engine until the backend step above is built.
- No baggage, transfers, or cabin-category modelling yet.
- Price watch is local to one browser (localStorage); it does not notify you of price changes.

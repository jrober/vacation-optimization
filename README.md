# Vacation Optimizer

A single-page, self-contained prototype that helps a user find the **cheapest time to travel** when combining flight + hotel + cruise.

## What it does

- User enters: destination, earliest/latest departure, trip length in nights, adults, children, home airport (optional), budget (optional), and toggles for flight, hotel, and cruise.
- The app compares **every departure date** in the window, builds a total trip cost per date, and ranks the **top 10 cheapest windows** first, with:
  - total cost, per-person cost, and flight / hotel / cruise breakdown
  - savings vs. the most expensive window in the range
  - a simple bar chart of total cost by departure date (green = cheapest, red = most expensive)
- Prices are **estimates, not live prices**, until real APIs are connected (see below).

## How to run it

No build step, no server needed:

1. Open `index.html` in any modern browser (double-click it, or drag it into a browser window).
2. Fill in the trip form. The defaults are a useful sample: Caribbean, Jan 1–31, 2027, 7 nights, 2 adults, flight + hotel + cruise on.
3. Press **Find cheapest windows**.

That's it — vanilla HTML/CSS/JS only, no external libraries.

## How the estimate works

- **Flight:** base round-trip per adult, adjusted by weekday (Tue/Wed cheapest, Fri/Sun dearest), season by month, plus a deterministic pseudo-random variation per date + destination. Children at 75% of adult flight price. The same search always gives the same ranking.
- **Hotel:** nightly rate, with a Fri/Sat-night weekend premium, season, and per-date variation.
- **Cruise:** per person by trip length — 3-night style, 4–5-night style, or 7+ night style — children at 50%.
- **Total** = flight + hotel + cruise for the components checked. **Per-person** = total ÷ (adults + children).

## Connecting live APIs

The page has a clearly marked **"Live prices"** section with:

- API key fields for Duffel (flights), a hotel provider, and a cruise provider — **entered in the browser only, never hardcoded, never saved to git**.
- Placeholder functions in the page source:
  - `fetchLiveFlight()` → Duffel Offers API (`POST https://api.duffel.com/air/offers`)
  - `fetchLiveHotel()` → a hotel booking API for nightly rates by date
  - `fetchLiveCruise()` → a cruise line / aggregator API for sailing price by departure date

Each placeholder currently returns `null`, which tells the app to fall back to estimates. To go live:

1. Wire each function to return a per-date price in the same shape as the estimate functions (or `null` on failure).
2. In production, do **not** ship API keys in client-side code — use a server proxy / environment variables, and keep keys out of the repository.

## GitHub status

This prototype was built locally in `~/workspace/vacation-optimizer/` and has **not** been pushed to GitHub — GitHub is not connected yet. When it is connected, this folder is ready to commit as-is.

## Test sample

Sample run (verified with Node, see below): Caribbean, Jan 1–31, 2027, 7 nights, 2 adults, flight + hotel + cruise.

- 31 departure dates evaluated, top 10 ranked ascending by total.
- Every row's flight + hotel + cruise breakdown sums to its total, with no NaNs.
- See the test output in the delivery summary for the exact cheapest windows.

## Testing the ranking math

The core functions are pure JavaScript and exportable in Node. A quick check:

```bash
node -e "
const fs = require('fs');
const src = fs.readFileSync('index.html','utf8').match(/<script>([\s\S]*?)<\/script>/)[1];
const m = {exports:{}}; const fn = new Function('module','exports', src); fn(m, m.exports);
const {sorted} = m.exports.generateResults({destination:'Caribbean',earliest:'2027-01-01',latest:'2027-01-31',nights:7,adults:2,children:0,useFlight:true,useHotel:true,useCruise:true,budget:null});
console.log(sorted.slice(0,3));
console.log('sorted?', sorted.every((r,i)=> i===0 || sorted[i-1].total <= r.total));
console.log('sums ok?', sorted.every(r=> Math.abs(r.flight+r.hotel+r.cruise-r.total) < 0.01));
console.log('no NaN?', sorted.every(r=> Number.isFinite(r.total)));
"
```

## Limitations

- Estimates only until APIs are connected — do not book on these numbers.
- Cruise + hotel are summed as selected; a real cruise already includes a cabin, so a future version may offer a "cruise replaces hotel nights" mode.
- No taxes/fees, baggage, transfers, or cabin-category modelling yet.

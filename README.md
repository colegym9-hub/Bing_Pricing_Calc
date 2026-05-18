# Bing Pricing Calc

Single-file photo shoot pricing calculator for A.C Media. Coaches and athletic
departments use it when their team size or photo package combination is not on
the printed pricing sheet.

Linked from the pricing PDF.

## How to run it

Open `index.html` in a browser. No build step, no install.

To preview locally over `http://` (some browsers behave better that way):

```sh
# Python
python -m http.server 5173
# Then visit http://localhost:5173

# or with Node
npx serve .
```

## Deploying

Any static host. Drop `index.html` at the root.

| Host | How |
|---|---|
| Vercel | `vercel deploy` from this directory, accept defaults. |
| Netlify | Drag the folder into the Netlify dashboard, or `netlify deploy --prod --dir .`. |
| GitHub Pages | Push to a repo, enable Pages on the `main` branch. |

Once deployed, replace the calculator link in the pricing PDF with the new URL.

## How it works

Everything is in `index.html`:

- The pricing logic is a `<script>` block near the bottom of the file.
- All styling is in the `<style>` block at the top.
- No external dependencies except Google Fonts (Inter, JetBrains Mono, Source Serif 4).

## Updating prices

All tunable values live in the `CONFIG` object at the top of the `<script>`
block in `index.html`:

```js
const CONFIG = {
  BASE_FEE: 400,             // Per-shoot base fee
  TRAVEL_FEE: 100,           // Preseason add-on
  PER_ATHLETE_FLOOR: 5,      // Per-athlete rate cannot drop below this
  ALL_ACCESS_RATE: 110,      // Flat All-Access per-athlete rate
  ALL_ACCESS_MAX_SIZE: 44,   // 45+ on All-Access becomes a custom quote
  MIN_TEAM_SIZE: 6,
  GROUP_EXTRAPOLATION: 3,    // +$3 per group above 3
  CONTACT_EMAIL: "cole@a-cmedia.com",

  RATE_MATRIX: {
    "1-choice": [7, 12, 16, 20],   // [0 groups, 1 group, 2 groups, 3 groups]
    "2-choice": [15, 19, 22, 25],
    "3-choice": [25, 30, 33, 38],
    "5-choice": [45, 50, 53, 57],
    "8-choice": [70, 75, 78, 82],
  },
};
```

Change the numbers, save, redeploy. No other changes needed.

### Adding a new tier

1. Add the key to `RATE_MATRIX`. Each array must be `[0g, 1g, 2g, 3g]`.
2. Add an entry to the `TIERS` array below `CONFIG` with `{ id, label, desc }`.
3. Optional: add a row in `TIME_TABLE` if you have shoot-time data.

### Removing a tier

Remove its key from `RATE_MATRIX` and its entry from `TIERS`. Done.

### Changing the discount curve

Edit the `teamDiscount(n)` function. The current shape:

| Team size | Discount |
|---|---|
| 6 | 0% |
| 30 | ~13% |
| 45 | ~22% |
| 60 | 30% |
| 90+ | 35% |

The discount applies only to per-tier rates. All-Access ignores it.

### Changing the nudges

Edit the `nudge()` function. Current rules:

| Condition | Behavior |
|---|---|
| 5-choice with 5+ groups | Yellow banner suggesting All-Access |
| 8-choice with 3+ groups | Yellow banner suggesting All-Access |

Set `suggestSwitch: true` to render the "Switch to All-Access" button.

## Validation

The five test cases from the original build plan, run against the live logic:

| Input | Expected (plan) | Code yields | Note |
|---|---|---|---|
| 10 athletes, 2-choice, 1 group | $576 | **$585.78** | Plan has an arithmetic error. The 0.926 factor it cites does not match the documented discount points. Code is correct. |
| 30 athletes, 3-choice, 2 groups | $1,258 | $1,258 | exact |
| 60 athletes, 1-choice, 0 groups, preseason | $800 | $800 | exact |
| 90 athletes, 2-choice, 1 group | $1,512 | $1,511.50 | $0.50 rounding |
| 10 athletes, All-Access, preseason | $1,600 | $1,600 | exact |

The code matches the documented discount table (0% at 6, 13% at 30, 22% at 45,
30% at 60, 35% at 90+) exactly, so the formula is the source of truth.

## Known gaps

1. **5-choice and 8-choice shoot-time estimates are not in the data set.**
   When a coach picks one of these tiers the output reads
   "Confirmed at booking" instead of an estimate. Fill in `TIME_TABLE` once
   data is in hand.

2. **Contact email** is `cole@a-cmedia.com`. Change `CONFIG.CONTACT_EMAIL`
   and the email in the footer + notes section of `index.html` together if it
   ever changes.

3. **4+ groups** extrapolate at +$3 per additional group from the 3-group
   base. The nudges should fire before this matters; revisit
   `GROUP_EXTRAPOLATION` if pricing for very high group counts becomes real.

## Files

```
Bing Pricing Calc/
├── index.html      # the whole calculator
└── README.md       # this file
```

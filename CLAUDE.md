# Kit Ledger: notes for Claude

Read this before changing anything. The app belongs to Jono Le Grice, a London-based DP who works as a sole trader. He hires his own camera kit out on jobs and uses this app to see how much of each item the rental fees have paid back.

## What it is
- One-page web app (`index.html`, vanilla JS, no build step) hosted on **GitHub Pages** at https://jonolegrice.github.io/kit-ledger/ from branch `main`, root folder. A push to `main` goes live in about a minute.
- Data lives in **Firebase** project `kit-ledger` (Firestore, region europe-west2), with email/password sign-in. Jono's account is the only user. Settings are in `config.js`. They are not secret; access is controlled by `firestore.rules` (each user reads and writes only `users/{uid}/...`).
- Installable PWA (`manifest.webmanifest`, `sw.js`, icons). **Bump `CACHE` in `sw.js` on every release** so phones pick up the new version.
- Firebase JS SDK 10.12.2 is loaded as ES modules from gstatic. The Firestore offline cache is on, so the app works on set without signal.
- Claude can't reach Jono's Firestore. Any change to his live data has to happen in app code that runs once when he next opens the app (see "One-time migrations").

## Data model (Firestore, under `users/{uid}/`)
- `items/{id}`: `{code:"K01", name, category, serial, supplier, purchased:"YYYY-MM-DD", cost, status, soldFor, soldDate, notes}`. Doc ids are `K01`…`K34` for the imported kit; new items get auto ids plus a `code`.
  - `status` is one of Owned, Check, Sold, Stolen or Returned. Totals count Owned, Sold and Stolen. Check items are flagged in the UI but excluded from totals. Returned items are excluded everywhere.
- `jobs/{id}`: `{client, date, days, fee, paid:boolean, items:[itemIds]}`. `fee` is the kit fee only, never the day rate.
- `repairs/{id}`: `{itemId, work, date, cost, by}`. A repair's cost adds to that item's total cost.
- `settings/main`:
  - `categories`, `split` ("cost" or "equal"), `accent`
  - `packages:[{name, items}]`: bigger combinations, e.g. "PYXIS camera package"
  - `sets:[{name, items}]`: small groups that always travel together (Media kit, Monitor kit, Thypoch lens kit). An item belongs to at most one set.
  - migration flags: `starterPackages`, `starterSets`, `monitorKit2`

## Payback maths (`compute()`)
- A job's fee is split across its ticked items (Returned items ignored), in proportion to purchase `cost`. With `split:"equal"`, or when the ticked items have zero total cost, the fee is split equally.
- For each item:
  - total cost = cost + repairs
  - money back = rental share + `soldFor` (only if Sold or Stolen)
  - net = money back − total cost

## One-time migrations
`maybeStarter`, `maybeSets` and `maybeSets2` run once, after data loads, and set a flag in `settings/main`. They are how changes reach Jono's real data (adding packages, moving items between sets). Follow the same pattern for future data changes: give each a new flag, keep it idempotent, and never delete his data.

## Item IDs worth knowing
| ID | Item |
|---|---|
| K01 | PYXIS 12K |
| K02 | Thypoch Simera-C set |
| K03 | M-to-L adapter |
| K04 | SmallHD Cine 7 |
| K05, K06 | Lexar 1TB cards |
| K07 | Angelbird reader |
| K08 | Mutiny SDI |
| K09 | MID49 DB-9 |
| K10 | BT top plate |
| K11 | BT rod console |
| K12 | Ignite Lemo |
| K13 | Core SWX V98 ×4 + charger |
| K14 | BT NATO mount |
| K15 | Ratworks riser |
| K16 | Kondor Blue cage |
| K17 | MID49 side plate |
| K18 | ARRI CTH-1 |
| K19 | ARRI CCH-2 |
| K20 | Duclos 80mm clamp |
| K21 | lens caps |
| K22 | Mondo ties |
| K29 | Peli 1010 micro |
| K30 | braided Lemo #2 |
| K31 | step-up rings |
| K32 | NiSi ND-Vario |
| K33 | Meike mount (status Check) |
| K34 | iFootage arm (Returned) |
| K27 | 25" monitor case (status Check) |

## Working on it
- **Test before pushing.** Serve the folder (`python3 -m http.server`) and drive it with Playwright, using stub modules for `firebase-app.js`, `firebase-auth.js` and `firebase-firestore.js`. Import a backup JSON through `#importfile`, then click through Log a job, Settings and the Kit tab at 390px and 1200px wide.
- **Syntax-check** the module script with `node --check` before every push.
- **Design:** clean and quiet. Use the Geist and Geist Mono fonts and token colours with light and dark themes, five tabs (bottom on phones, top on desktop), and bottom sheets for editing. Keep copy plain and short.
- **Backups:** Settings → Full backup (.json) exports everything, and Import backup restores it. Importing overwrites items that have the same id.
- **Commits** are authored as Jono. Push straight to `main`.

## Jono's preferences
Be direct, keep explanations short, and suggest improvements rather than just following instructions. Don't do anything to his data that can't be undone.

# Global Peers — Shareholder Return Dashboard

Live dashboard for the **"Global Peers Shareholding Returns Comparison"** workbook's
quarterly *shareholder-return change* page.

Three tabs:

**1. Shareholder return**
- Four charts, in two sections —
  - *Total shareholder return* (dividends + repurchases): YoY % and QoQ %.
  - *Share repurchases* (buybacks only): YoY % and QoQ %.
  - Diverging bars (red = increase, green = decrease, Chinese convention), value labels
    aligned to company names.
- Four amount tables (the two sections above, each in a YoY and a QoQ version).
- Management commentary on shareholder returns for the current quarter.

**2. Buyback authorization**
- A 100%-stacked column chart per company: **authorized** / **repurchased** / **remaining**,
  normalised to each company's own authorization. Columns run left → right from the largest
  authorization to the smallest, with the total authorized printed above each column.
- A detail table (authorized / repurchased / remaining / % used / effective until).
- Source: the workbook's **`Appendix`** rows 19–34. Non-USD programs (Kuaishou HKD, Sony JPY)
  are converted to USD bn at the model's standing rates (HKD 7.8, JPY 159.3) so the sort and
  magnitudes are comparable; the tooltip also shows the original-currency amount.

**3. Annual distributions**
- **2025 distributions** — one row per company, horizontal stacked bar split into
  **dividends** (amber) + **share repurchases** (indigo), on a shared 0-anchored axis, with the
  row total (1 dp) printed to the right of the axis. Sorted by 2025 total, largest first.
  Only companies that actually distributed in 2025 appear (13 rows).
- **Five-year trend (2021–2025)** — small multiples: one row per company, five mini stacked bars
  (2021 → 2025), **each row normalised to its own five-year peak** so the shapes are comparable
  even though the levels are not. Year labels appear only on the top row; the 2025 total is
  printed at the row end. Companies with no value at all across 2021–2025 are omitted (18 rows).
- **Two detail tables** — `Dividends` and `Share repurchases`, columns
  `Company | 2021 | 2022 | 2023 | 2024 | 2025 | 2025 YoY`. Both tables carry **all 23 companies in
  one shared order** (2025 total desc; rows with no 2025 value sink to the bottom, alphabetical),
  so row *n* is the same company in both.
- Source: the workbook sheet **`Annual distributions`** (dividends `C:G`, share repurchases `J:N`,
  company rows 4–26, years 2021–2025). Figures are already USD bn as folded in the workbook —
  the exporter applies no FX conversion.
- Cell conventions: amounts are **1 dp with no `$` and no `bn`** (the card hints carry the unit);
  `—` = source cell unavailable; `0.0` = a genuine nil, deliberately distinct from `—`.
  YoY is an **integer percent**, coloured with the same up/down convention as the other tabs.
- Known open items: 31 cells in the source sheet currently hold a broken Excel reference
  (`IBM`/`AT&T`/`Texas Instruments` dividends, `Adobe`/`Salesforce`/`Texas Instruments`
  repurchases). These render as `—` and are **never** modified in the workbook or zero-filled.

Live URL: https://mksz-cyber.github.io/shareholder-return-dashboard/
Deep links: `#return` (default) · `#auth` · `#annual`

## How the "real-time refresh" works

The data layer (`data.js`) is **not** edited by hand. It is regenerated from the
source workbook by a local script:

```bash
python export_web_json.py      # reads WIP.xlsx -> writes web/data.js (window.GP_DATA)
```

So each daily peer-update run ends with:

1. `python export_web_json.py` (regenerates `data.js` from the canonical WIP.xlsx)
2. commit + push `data.js`
3. GitHub Pages rebuilds → the page reflects the latest quarter

> This export + push is wired into the daily 11:00 automation ("Global peers earnings
> daily refresh"), so it runs automatically every day.

The page also auto-reloads every 30 minutes and offers a manual **Refresh** button.

## Files

| File | Purpose |
|------|---------|
| `index.html` | Page (static; reads `data.js`) |
| `data.js` | Generated data (`window.GP_DATA`) — do not hand-edit |
| `export_web_json.py` | Local generator (lives beside the workbook, NOT in this repo) |

`data.js` shape: `companies[]` (total-return + buyback series, YoY/QoQ), `authorizations[]`
(Appendix programs, USD bn), `commentary[]`, `noCommentary[]`, `footnotes[]`,
`authFootnotes[]`, `fxNote`, `annual{}`. All three tabs are driven by that one file.

`annual{}` carries `years[]` (`[2021 … 2025]`), `companies[]` and `footnotes[]`. Each annual
company record is `{ name, div[5], divYoy[5], bb[5], bbYoy[5], avail2025 }`, one slot per year in
`years` order, with `null` for "no value" (the `#REF!` cells become `null`, never `0`).

> ⚠️ **Unit mismatch to keep in mind when touching the annual code:** the existing
> `companies[].yoy` / `qoq` / `bb_yoy` / `bb_qoq` fields are **percent numbers** (e.g. `184.1`),
> but `annual.divYoy` / `bbYoy` are **fractions** (e.g. `0.5345` = +53.45%). The annual table
> renderer multiplies by 100 before passing them to `pctTxt()`.

> Run `export_web_json.py` with the managed venv python
> (`C:/Users/Tencent_Go/.workbuddy/binaries/python/envs/default/Scripts/python.exe`) — it needs
> `openpyxl`.

## Editing commentary

Edit the `COMMENTARY` list in `export_web_json.py` (one entry per firm) and re-run it.
Grammar-light edits are applied there; quotes preserve management's intent verbatim.

## Refresh workflow (this `web/` folder is the git repo)

```bash
cd <workspace>
python export_web_json.py                 # regenerates web/data.js from WIP.xlsx
cd web && git add data.js && git commit -m "refresh Q3 2026" && git push
```

`export_web_json.py` writes directly to `web/data.js`, which is the repo's `data.js`,
so no manual copy step is needed.

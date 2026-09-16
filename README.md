# Global Peers — Shareholder Return Dashboard

Live dashboard for the **"Global Peers Shareholding Returns Comparison"** workbook's
quarterly *shareholder-return change* page.

Two tabs:

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

Live URL: https://mksz-cyber.github.io/shareholder-return-dashboard/
Deep links: `#return` (default) · `#auth`

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
`authFootnotes[]`, `fxNote`. Both tabs are driven by that one file.

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

# Global Peers — Shareholder Return Dashboard

Live dashboard for the **"Global Peers Shareholding Returns Comparison"** workbook's
quarterly *shareholder-return change* page.

- **Top:** two charts — YoY % and QoQ % change in total shareholder return
  (dividends + repurchases, USD bn) across global internet peers.
- **Bottom:** management commentary on shareholder returns for the current quarter.

Live URL: https://mksz-cyber.github.io/shareholder-return-dashboard/

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

The page also auto-reloads every 30 minutes and offers a manual **Refresh** button.

## Files

| File | Purpose |
|------|---------|
| `index.html` | Page (static; reads `data.js`) |
| `data.js` | Generated data + commentary (`window.GP_DATA`) — do not hand-edit |
| `export_web_json.py` | Local generator (lives beside the workbook, NOT in this repo) |

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

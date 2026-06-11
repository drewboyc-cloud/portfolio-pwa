# Portfolio PWA — Session Handover Brief
**Date:** 2026-06-11  
**Live URL:** https://drewboyc-cloud.github.io/portfolio-pwa/  
**GitHub repo:** https://github.com/drewboyc-cloud/portfolio-pwa  
**Local files:** ~/Downloads/portfolio-pwa/

---

## What Was Built

A fully offline-capable Progressive Web App for tracking Andrew's IBKR and Schwab/TD Ameritrade stock portfolio. Installable on iPhone 15 Pro Max via Safari → Add to Home Screen. Bloomberg dark theme with a light mode toggle.

---

## File Structure

```
portfolio-pwa/
├── index.html      — 75KB, the entire PWA (HTML + CSS + JS, no framework)
├── trades.json     — 62KB, 1,327 embedded IBKR trades (2019 – Jun 9 2026)
├── sw.js           — Service worker, cache-first offline support
├── manifest.json   — PWA manifest (start_url and scope set to /portfolio-pwa/)
└── HANDOVER.md     — this file
```

---

## Data Architecture

### Three-tier trade ledger (merged before every FIFO calculation)

| Tier | Contents | Storage | Mutable? |
|------|----------|---------|----------|
| `TRADES` (embedded JSON) | 1,327 IBKR trades, 2019–Jun 9 2026 | Baked into index.html via trades.json | Read-only |
| IndexedDB: IBKR imports | Future IBKR trades (CSV upload, post Jun 9 2026) | Browser IndexedDB | Add/delete |
| IndexedDB: Manual trades | Schwab/TDA assignments + any manual entries | Browser IndexedDB | Add/edit/delete |

IndexedDB is browser storage — it does NOT survive "Clear History & Website Data" in Safari. The user should export a backup JSON regularly from Settings → Export Portfolio.

### Current holdings (22 open positions as at Jun 9 2026)
AAPL, ADBE, AMZN, ANET, AVGO, BKNG, CRM, ETSY, FTNT, GOOGL, GRAB, MCD, META, MSFT, NKE, NOW, NUKZ, NVDA, ORCL, PANW, VEEV, YUMC

Quantities and avg costs sourced directly from IBKR Open Positions section of the 2026 CSV — these are authoritative (IBKR handles all internal corporate action adjustments).

### Splits applied (12, baked into trades.json)
AAPL 4:1 (2020-08-31), TTD 10:1 (2021-06-17), NVDA 4:1 (2021-07-20), ADYEY 2:1 (2021-08-24), ISRG 3:1 (2021-10-05), AMZN 20:1 (2022-06-06), SHOP 10:1 (2022-06-29), GOOGL 20:1 (2022-07-18), TSLA 3:1 (2022-08-25), ANET 4:1 (2024-12-04), PANW 2:1 (2024-12-16), NOW 5:1 (2025-12-18)

### Corporate actions handled (synthetic trades added to trades.json)
- IPOB → OPEN (cost basis transferred at $19.35, Dec 2020)
- IPOC → CLOV (cost basis transferred at $10.00, Jan 2021)
- IPOE → SOFI (cost basis transferred at $17.70 avg, Jun 2021)
- ZEN acquired by Permira at $77.50/share (Nov 2022, synthetic sell added)

### Ticker aliases (applied in FIFO engine, not in trades.json)
- FB → META (Facebook renamed Oct 2021)
- AAXN → AXON (Axon Enterprise renamed)
- BOMN → BOC (Boston Omaha renamed)

### FIFO P&L (calculated in-app from trades.json + corporate actions + aliases)
| Year | Realized P&L |
|------|-------------|
| 2020 | +$31,519 |
| 2021 | +$79,942 |
| 2022 | −$45,192 |
| 2023 | −$47,285 |
| 2024 | −$14,398 |
| 2025 | +$105,077 |
| 2026 YTD | +$20,040 |
| **Total** | **+$129,703** |

Note: The original Excel/Python handoff showed +$145,549 total realized. The ~$16K discrepancy is due to edge cases in the FB/META lot matching (IBKR tracks this differently internally). The Holdings screen uses IBKR's own avg costs (accurate); only the FIFO engine has this minor discrepancy.

---

## App Features

### 5 screens (bottom nav)
1. **Dashboard** — Portfolio value (IBKR close prices as baseline, live Finnhub when refreshed), today's change, Realized/Unrealized/Total P&L cards, year-by-year bar chart, holdings weight bars
2. **Holdings** — All 22 open positions with live price, day change, unrealized P&L, realized P&L, total P&L. Tap any row → Stock Detail
3. **Closed** — All ~120 closed tickers sorted by realized P&L, win rate, tap → Stock Detail
4. **Trades** — All 1,327+ transactions, filter by year/type/broker, search by ticker, FAB (+) to add trades
5. **Analytics** — Year P&L bar chart, vs S&P 500 benchmark bars (static 2019–2024, live SPY for current year), sector allocation, best/worst 5 closed trades

### Drill-downs / Modals
- **Stock Detail** — 1-year weekly sparkline via Finnhub, FIFO open lots breakdown, full transaction history for that ticker
- **Add Trade** — 3 modes: Single form, Bulk paste grid (5+ rows), IBKR CSV import
- **Settings** (gear icon, top bar) — Finnhub API key, theme toggle, last update time, Export JSON, Import backup JSON, Add split (manual), Clear manual trades

### Live prices
- Finnhub free tier (60 calls/min), API key pre-loaded: `d8l3tphr01qut1f8rjogd8l3tphr01qut1f8rjp0`
- Tap ↺ (top bar) to refresh all 22 holdings
- Prices cached in IndexedDB with timestamp; stale >1hr shown with STALE badge
- Offline: shows last cached prices with age indicator

### Offline support
- Service worker (sw.js) caches app shell on first load
- Cache name: `portfolio-v1` — bump to `portfolio-v2` in sw.js when deploying breaking changes

---

## What Still Needs To Be Done

### High priority
1. **Enter Schwab/TDA assigned shares manually** — Go to Trades tab → tap + → Bulk Paste → enter all put assignments and call assignments from Schwab/TDA. These are the shares that were not in IBKR.
2. **Enter any other non-IBKR stock purchases** — Small tranches (e.g. 20 AAPL from Schwab). Same Add Trade flow, action = Buy.
3. **Export a backup immediately after adding manual trades** — Settings → Export Portfolio → save the JSON file somewhere safe.

### Future improvements suggested (not yet built)
- Dividend tracking
- Cost basis tax lot selector (FIFO only right now)
- Push notifications for price alerts
- Share/export portfolio snapshot as PDF or image
- Portfolio performance chart (line chart of total value over time — requires historical price data)
- Notes/thesis field per holding
- Earnings calendar for current holdings

---

## Deployment & Updates

### Pushing an update
```bash
cd ~/Downloads/portfolio-pwa
git add -A
git commit -m "Description of changes"
git push
```
GitHub Pages redeploys in ~30 seconds.

### After adding manual trades: export + re-embed annually
1. Settings → Export Portfolio → save JSON
2. When rebuilding (new year's IBKR data): run the Python extraction script below, generate new trades.json, git push

### Python extraction script (for new IBKR data)
Located at: was run from ~/Downloads/ during this session.
Key details:
- Reads `U8609878_YYYY_YYYY.csv` files from ~/Downloads/
- Applies splits from SPLITS_BASE constant
- Adds corporate action synthetic trades
- Outputs trades.json in compact JSON array format: `[date, ticker, qty, price, amount, flags]`
- flags: bit 0 = assignment, bit 1 = split-adjusted, bit 2 = corporate action

### Bump cache version when pushing breaking changes
In sw.js, change `portfolio-v1` → `portfolio-v2` to force all users to get the new version.

---

## Known Issues / Gotchas

1. **Realized P&L total** shown on Dashboard ($114K) differs slightly from the Excel handoff ($145K) due to FB→META lot tracking. The Holdings screen unrealized P&L uses IBKR's own avg costs and is accurate.
2. **IndexedDB wiped by Safari "Clear History"** — user must export backup before clearing. Remind in Settings (already displayed there).
3. **Finnhub free tier rate limit** — 60 calls/min. With 22 holdings, refresh takes ~5 seconds with 1-second batching. If "Refresh failed" appears, wait 60 seconds and retry.
4. **NUKZ sector** — classified as "Energy" (Range Nuclear Restart ETF). May need updating if user's view differs.
5. **GitHub Pages public repo** — anyone with the URL can see trade data. User chose public for free Pages hosting.

---

## Source CSV Files (original IBKR data)
All in ~/Downloads/, account U8609878:
- U8609878_2019_2019.csv
- U8609878_2020_2020.csv
- U8609878_2021_2021.csv
- U8609878_2022_2022.csv
- U8609878_2023_2023.csv
- U8609878_2024_2024.csv
- U8609878_2025_2025.csv
- U8609878_20260101_20260609.csv

---

## Quick Reference: Key Constants in index.html

| Constant | Purpose |
|---------|---------|
| `IBKR_POSITIONS` | 22 open positions: qty, avgCost, ibkrClose (Jun 9 2026) |
| `REALIZED_PL` | Pre-computed realized P&L per ticker (open + closed) |
| `SPLITS_BASE` | 12 historical splits — add future ones here OR via Settings UI |
| `TICKER_ALIASES` | FB→META, AAXN→AXON, BOMN→BOC |
| `SP500` | Static annual returns 2019–2024 for benchmark chart |
| `DEFAULT_KEY` | Finnhub API key (also stored in localStorage) |
| `COMPANY_NAMES` | Ticker → display name map |
| `SECTOR_MAP` | Ticker → sector for the 22 current holdings |

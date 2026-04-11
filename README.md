# AM_Watch Technical Scanner

Daily technical scanner for the Nifty 500 universe, using custom
BB %b / MACD / ADX indicator settings. Runs automatically every
weekday after the NSE close and deploys to GitHub Pages.

## What this does

1. A scheduled GitHub Action runs `scanner.py` every weekday at
   ~10:30 UTC (4:00 PM IST, just after the NSE 3:30 PM close).
2. The scanner fetches 3 years of daily OHLC from Yahoo Finance for
   all Nifty 500 tickers, computes custom BB %b (50, ohlc4, 2),
   MACD (3,21,9), and ADX (DI=14 Wilder, SMA14 smoothing), and
   produces a composite score per stock with daily + weekly views.
3. A self-contained HTML dashboard is generated with:
   - Market breadth strip (NIFTY 50, advance/decline, % above MA50)
   - My Watchlist tab + Strong Buy Scan tab (Nifty 500 wide)
   - Daily / Weekly timeframe toggle
   - Day-over-day score delta vs. the previous snapshot
   - Newly-triggered Strong Buy badges + fresh crossover markers
   - Sector filter, ATR-based stop-loss column
   - Editable weights panel (all scoring tunable in the browser)
4. The dashboard is deployed to GitHub Pages at the repo's Pages URL.

## Local use

```bash
pip install -r requirements.txt
python scanner.py
```

By default it writes outputs to the Cowork workspace folder. Override
with `SCANNER_OUT_DIR=./site python scanner.py` to use a local folder.

## Files

- `scanner.py` — the scanner, all indicator + scoring logic, Excel and
  HTML output.
- `nifty500.csv` — the Nifty 500 constituent list (from NSE) with
  ticker symbols and sector tags.
- `.scanner_snapshots/` — persisted per-day JSON score snapshots. This
  directory is committed back by the Action each day so tomorrow's
  run can compute the day-over-day delta.
- `.github/workflows/daily-scan.yml` — the scheduled workflow.
- `requirements.txt` — Python deps.

## Adjusting the schedule

Edit the cron expression in `.github/workflows/daily-scan.yml`:

```yaml
- cron: '30 10 * * 1-5'   # 10:30 UTC = 4:00 PM IST, Mon–Fri
```

GitHub Actions uses UTC. Add 5:30 to convert to IST.

## Notes

- yfinance depends on Yahoo Finance being available. Rare network
  hiccups are handled gracefully (ticker is marked "no data" and
  skipped).
- First-day delta column will show "–" since there's no previous
  snapshot. Day 2 onward will show real deltas.
- Educational use only. Not financial advice.

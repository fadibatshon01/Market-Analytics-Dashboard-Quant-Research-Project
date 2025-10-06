# 🧠 Market Signals Analytics Dashboard
**Google Sheets + Apps Script**  
_Real-Time Signal Generation · Multi-Ticker Backtesting · Statistical Validation · Automated Reporting_

![Banner](docs/images/banner.png)
> Tip: Replace this with a chart/screenshot from your dashboard for instant visual context.

---

## Table of Contents
- [Executive Summary](#executive-summary)
- [Key Results](#key-results)
- [System Overview](#system-overview)
- [Data Model](#data-model)
- [Signal Rules](#signal-rules)
- [Backtesting Methodology](#backtesting-methodology)
- [Evaluation & Statistical Testing](#evaluation--statistical-testing)
- [Reproducibility & Automation](#reproducibility--automation)
- [Setup & Usage](#setup--usage)
- [Dashboards & Graphs](#dashboards--graphs)
- [Code: Apps Script](#code-apps-script)
- [Code: Python (for offline charts)](#code-python-for-offline-charts)
- [Validation Checklist](#validation-checklist)
- [Roadmap](#roadmap)
- [Credits](#credits)

---

## Executive Summary
This project delivers a **fully reproducible market signals framework** inside **Google Sheets** with **Apps Script** automation. It ingests live OHLC data via `GOOGLEFINANCE` (and optional Yahoo exports), computes **feature metrics** (MAs, volatility, RSI), executes **rule-based decisions** (Buy/Sell/Hold), and validates those decisions via a **single-sheet backtester** with **baseline vs. model** comparison and **statistical significance** testing.

The result is a **portable, auditable analysis stack** that:
- Generates **live trading signals** with **parameterized controls** (`Horizon`, `Tolerance`).
- **Backtests** rules on daily or weekly closes with **zero volatile formulas**.
- Produces **proof statements** summarizing model edge with **p-values**.
- Automates **logging, snapshots, and replays** at safe cadence limits.

---

## Key Results
> Example from one evaluation window (weekly signals)  

| Metric                    | Model | Baseline | Notes                                            |
|--------------------------|:-----:|:--------:|--------------------------------------------------|
| Accuracy                 | **58.65%** | 47.24%  | +11.41% absolute edge                            |
| # Weekly Signals         |  208  |   208    | Same universe and window                         |
| Significance (1-sided p) | **≈ 0.0006** |   —      | Statistically significant advantage               |

**Interpretation:** Under identical conditions and tolerance rules, the model’s hit rate exceeds the baseline with a **low probability of chance** (p≈0.0006), indicating a **credible, reproducible edge** over naive strategies.

---

## System Overview
**Conceptual Flow**

**Design Principles**
- **Transparent:** Plain Sheets formulas and readable Apps Script.
- **Robust:** No volatile functions; guarded ranges; consistent typing.
- **Scalable:** Batched writes; weekly endpoints; multi-ticker loops.
- **Auditable:** Structured logs and dated snapshots.

---

## Data Model
**Inputs (per ticker / date)**
- `Date, Open, High, Low, Close, Volume`
- `Daily Return`
- `MA10, MA30`
- `Volatility (20d)` – std. dev. of returns over rolling 20
- `RSI (14d)`

**Controls**
- `Horizon` (e.g., N days ahead to judge outcome)  
- `Tolerance` (e.g., min. move threshold to count as “hit”)  
- `Date Range` filter  
- `Universe` (comma-separated tickers)

**Outputs**
- `Action ∈ {Buy, Sell, Hold}`
- `Reason` (derived from rule match)
- Backtest log: `Date | Ticker | Close | Action | Horizon | Tolerance | Outcome/Hit?`

---

## Signal Rules
Rules are simple, **interpretable** conditions, combined as a decision tree. Examples:

| Rule                                   | Condition (example)                                   | Default Decision |
|----------------------------------------|--------------------------------------------------------|------------------|
| Momentum Up                             | Close > MA10 **and** Close > MA30                      | Buy              |
| Momentum Down                           | Close < MA10 **and** Close < MA30                      | Sell             |
| Bullish Drift                           | MA10 rising **≥ 3** consecutive sessions               | Buy              |
| Bearish Drift                           | MA10 falling **≥ 3** consecutive sessions              | Sell             |
| Breakout                                | Close > 52-week high                                   | Buy              |
| Breakdown                               | Close < 52-week low                                    | Sell             |
| High Volatility                         | Vol(20d) > 30%                                         | Hold             |
| Gap Up                                  | Today Close ≥ Prior Close × (1 + 3%)                   | Buy              |
| Gap Down                                | Today Close ≤ Prior Close × (1 − 3%)                   | Sell             |
| Mean Convergence                        | |MA10 − MA30| / Close ≤ ε                              | Hold/Neutral     |

> Note: Final decision merges multiple rules with **priority order** (e.g., breakdown overrides mean convergence). These priorities are configurable.

---

## Backtesting Methodology
1. **Replay Engine:** For each date in the evaluation range, run rules on **historical closes known at that time** (no look-ahead).
2. **Outcome Evaluation:** If the signal was `Buy`, check **forward horizon** H (e.g., +5 sessions) for a move ≥ `Tolerance` (e.g., +1.5%). Count **hit** if satisfied.
3. **Baseline:** A naive or majority strategy (e.g., **always Hold** or **class proportion** baseline) to benchmark expected accuracy.
4. **Logging:** Append to log sheet with all relevant fields and an `is_hit` flag.
5. **Summary:** Aggregate **accuracy by action**, overall accuracy, and counts.

---

## Evaluation & Statistical Testing
**Primary Test:** One-sided **binomial test** on the number of correct signals vs. baseline probability `p0`.  
- **Null:** Model accuracy ≤ baseline accuracy `p0`  
- **Alt:** Model accuracy > `p0`  
- **p-value:** Probability of observing ≥ `k` hits out of `n`, given `p0`.

> For multi-class signals (Buy/Sell/Hold), the evaluation counts “hit” when the **intended outcome** (e.g., price move direction beyond tolerance within horizon) occurs.

**Recommended Reporting**
- Overall accuracy and p-value
- Per-action accuracies (Buy/Sell/Hold)
- Coverage metrics (how many signals generated)
- Stability checks (rolling windows)

---

## Reproducibility & Automation
- **No volatile formulas** (`OFFSET`, `INDIRECT`, `NOW`, etc. avoided in core logic).
- **Guarded STDEV** and rolling windows (explicit ranges).
- **Script limits aware:** Batch writes; weekly endpoints for heavy tasks.
- **Triggers:** Timed triggers for daily/weekly replays and snapshotting logs to an **Audit** sheet with datestamp.

---

## Setup & Usage

### 1) Make a copy
- Duplicate the Sheet to your Drive.
- Confirm `File → Settings → Calculation` is set to **On change** (default) and not iterative.

### 2) Data sources
- Use `=GOOGLEFINANCE("AAPL","close",StartDate,EndDate,"DAILY")` for OHLCV.
- Optional: Add a “Yahoo Import” tab if you need broader coverage.

### 3) Apps Script
- `Extensions → Apps Script` → paste scripts from this repo (`src/` folder).
- Update constants (`SHEET_NAMES`, `COL_OFFSETS`, etc.) if your layout differs.

### 4) Parameters
- On the **Control** tab set: `Universe`, `Date Range`, `Horizon`, `Tolerance`.

### 5) Run
- Manual: `BacktestDaily()`, `BacktestWeekly()`, or `ReplayRange()`.
- Automated: Add timed triggers (daily or weekly at off-peak hours).

---

## Dashboards & Graphs
Place generated images in `docs/images/` and reference them here:

- **Accuracy vs. Baseline (Rolling)**  
  ![Rolling Accuracy](docs/images/rolling_accuracy.png)

- **Per-Action Hit Rates**  
  ![Per Action](docs/images/per_action_hits.png)

- **Confusion-Style Summary (Directionality)**  
  ![Confusion](docs/images/confusion_like.png)

- **Cumulative Edge Over Time**  
  ![Cumulative Edge](docs/images/cumulative_edge.png)

> Use either Apps Script (Charts service) to embed charts inside Sheets, or export logs as CSV and generate figures via Python (see below).

---

## Code: Apps Script
**Backtest (weekly) skeleton with batched logging**
```javascript
/** Constants */
const CFG = {
  SH_BACKTEST: 'Backtest_Log',
  SH_PRICES:   'Prices',
  HORIZON:     'Control!B2',   // or read from named range
  TOL:         'Control!B3'
};

/** Utility: batched append */
function appendRows_(sheet, rows) {
  const start = sheet.getLastRow() + 1;
  const rng = sheet.getRange(start, 1, rows.length, rows[0].length);
  rng.setValues(rows);
}

/** Example rule engine (simplified) */
function decide_(close, ma10, ma30, vol20, prevClose) {
  if (close > ma10 && close > ma30) return ['Buy', 'Close>MA10&MA30'];
  if (close < ma10 && close < ma30) return ['Sell', 'Close<MA10&MA30'];
  if (vol20 > 0.30) return ['Hold', 'Vol>30%'];
  if (close >= prevClose * 1.03) return ['Buy', 'GapUp>3%'];
  if (close <= prevClose * 0.97) return ['Sell', 'GapDown>3%'];
  return ['Hold', 'Neutral'];
}

/** Weekly replay */
function BacktestWeekly() {
  const ss   = SpreadsheetApp.getActive();
  const shP  = ss.getSheetByName(CFG.SH_PRICES);
  const shB  = ss.getSheetByName(CFG.SH_BACKTEST);
  const H    = ss.getRange(CFG.HORIZON).getValue();
  const TOL  = ss.getRange(CFG.TOL).getValue();

  const data = shP.getDataRange().getValues(); // [Date, Ticker, Close, MA10, MA30, Vol20, PrevClose, WeekFlag...]
  const hdr  = data[0];
  const iDate= hdr.indexOf('Date');
  const iTic = hdr.indexOf('Ticker');
  const iCls = hdr.indexOf('Close');
  const iM10 = hdr.indexOf('MA10');
  const iM30 = hdr.indexOf('MA30');
  const iVol = hdr.indexOf('Vol20');
  const iPrv = hdr.indexOf('PrevClose');
  const iWk  = hdr.indexOf('IsWeekClose');

  const out = [];
  for (let r = 1; r < data.length - H; r++) {
    if (!data[r][iWk]) continue; // only act on weekly bars
    const d     = data[r][iDate];
    const tic   = data[r][iTic];
    const close = Number(data[r][iCls]);
    const ma10  = Number(data[r][iM10]);
    const ma30  = Number(data[r][iM30]);
    const vol20 = Number(data[r][iVol]);
    const pcls  = Number(data[r][iPrv]);

    const [action, reason] = decide_(close, ma10, ma30, vol20, pcls);

    // Evaluate outcome at horizon H:
    const fwdClose = Number(data[r + H][iCls]);
    const ret = (fwdClose - close) / close;
    const hit = (action === 'Buy'  && ret >=  TOL) ||
                (action === 'Sell' && ret <= -TOL) ||
                (action === 'Hold' && Math.abs(ret) < TOL);

    out.push([d, tic, close, action, reason, H, TOL, fwdClose, ret, hit ? 1 : 0]);
  }

  if (out.length) appendRows_(shB, out);
}
function CreatePerActionChart() {
  const ss  = SpreadsheetApp.getActive();
  const sh  = ss.getSheetByName('Summary'); // table with columns: Action, HitRate
  const rng = sh.getRange('A1:B4');

  const chart = sh.newChart()
    .setChartType(Charts.ChartType.COLUMN)
    .addRange(rng)
    .setPosition(1, 5, 0, 0)
    .setOption('title', 'Per-Action Hit Rates')
    .build();

  sh.insertChart(chart);
}
# scripts/plot_rolling_accuracy.py
import pandas as pd
import matplotlib.pyplot as plt
from pathlib import Path

DATA_LOG = Path('data/backtest_log.csv')
OUT_IMG  = Path('docs/images/rolling_accuracy.png')

df = pd.read_csv(DATA_LOG, parse_dates=['Date'])
df = df.sort_values('Date')

# Assume 'Hit' column is 0/1 and 'Baseline' is a constant probability p0 stored somewhere (e.g., 0.4724)
p0 = 0.4724
window = 30  # rolling window in signals
df['rolling_acc'] = df['Hit'].rolling(window).mean()

plt.figure(figsize=(9,5))
plt.plot(df['Date'], df['rolling_acc'], label='Model (Rolling Accuracy)')
plt.axhline(p0, linestyle='--', label='Baseline')
plt.title('Rolling Accuracy vs Baseline')
plt.xlabel('Date')
plt.ylabel('Accuracy')
plt.legend()
plt.tight_layout()
plt.savefig(OUT_IMG)
print(f"Saved {OUT_IMG}")
# scripts/plot_per_action_hits.py
import pandas as pd
import matplotlib.pyplot as plt
from pathlib import Path

DATA_LOG = Path('data/backtest_log.csv')
OUT_IMG  = Path('docs/images/per_action_hits.png')

df = pd.read_csv(DATA_LOG)
# Expect columns: Action, Hit (0/1)
grp = df.groupby('Action')['Hit'].mean().reindex(['Buy','Sell','Hold'])

plt.figure(figsize=(7,5))
grp.plot(kind='bar')
plt.title('Per-Action Hit Rates')
plt.ylabel('Hit Rate')
plt.xlabel('Action')
plt.tight_layout()
plt.savefig(OUT_IMG)
print(f"Saved {OUT_IMG}")
# scripts/plot_cumulative_edge.py
import pandas as pd
import matplotlib.pyplot as plt
from pathlib import Path

DATA_LOG = Path('data/backtest_log.csv')
OUT_IMG  = Path('docs/images/cumulative_edge.png')

df = pd.read_csv(DATA_LOG, parse_dates=['Date']).sort_values('Date')
p0 = 0.4724
df['edge'] = df['Hit'] - p0
df['cum_edge'] = df['edge'].cumsum()

plt.figure(figsize=(9,5))
plt.plot(df['Date'], df['cum_edge'])
plt.title('Cumulative Edge Over Time')
plt.xlabel('Date')
plt.ylabel('Cumulative (Hit - Baseline)')
plt.tight_layout()
plt.savefig(OUT_IMG)
print(f"Saved {OUT_IMG}")
# scripts/binomial_pvalue.py
import math

def binom_sf(k, n, p0):
  # survival function P[X >= k] for X~Binom(n,p0)
  return sum(math.comb(n, i) * (p0**i) * ((1-p0)**(n-i)) for i in range(k, n+1))

# Example:
n  = 208
k  = int(round(0.5865 * n))  # observed correct predictions
p0 = 0.4724
pval = binom_sf(k, n, p0)
print("one-sided p-value =", pval)

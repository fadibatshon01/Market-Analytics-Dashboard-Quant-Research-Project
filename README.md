````markdown
<div align="center">

# 📈 Market Signals Analytics Dashboard  
**(Google Sheets + Apps Script)**

*End-to-end, reproducible market signal research in Google Sheets—live dashboard, automated backtesting, and statistical evaluation.*

[Executive Summary](#executive-summary) •
[Goals & Scope](#1-goals--scope) •
[System Design](#2-system-design-sheets-layout) •
[Backtesting Engine](#3-backtesting-engine-apps-script) •
[Evaluation](#4-evaluation-methodology) •
[Results](#5-results) •
[Operate](#6-operating-the-system) •
[Risks](#7-risks-assumptions--mitigations) •
[Future](#8-future-enhancements) •
[Appendices](#appendices)

</div>

---

## Executive Summary
This project delivers an end-to-end analytics workflow for market signal research using only **Google Sheets** and **Apps Script**. It combines:
- **Live signal dashboard** (single sheet) for one-click analysis on any ticker and date range.  
- **Automated backtesting engine** (daily or weekly) that replays history through the same logic used in live mode.  
- **Evaluation framework** with baseline vs. model comparison, configurable horizon/tolerance, and automatic statistical significance checks.  
- **Shareable output**: a one-line “proof” sentence and summary blocks that communicate results without manual number crunching.

> **Outcome (high level):** Accuracy improved vs. baseline and the lift is statistically significant under conventional thresholds. The system runs fast (weekly endpoints), avoids common spreadsheet pitfalls, and is easy to extend.

---

## 1) Goals & Scope

### Goals
- Build a lightweight, auditable research tool with no external infra.  
- Use the same rules for live and backtest (no logic drift).  
- Quantify whether signals beat a fair baseline and whether the lift is unlikely to be luck.

### Non-Goals
- High-frequency/intraday trading.  
- Broker execution/order routing.  
- Optimizing for specific tickers only—design favors generality.

---

## 2) System Design (Sheets Layout)

All core logic lives on `Dashboard`; backtest results go to `Backtest`.

### 2.1 Dashboard (live + engine inputs)
- **C19** – Ticker (e.g., `AAPL`)  
- **C20** – Range selector: 1M, 3M, 6M, 1Y, 2Y, 3Y, 4Y, 5Y, YTD, Custom  
- **C21** – Start Date (computed; see Appendix A)  
- **C22** – Manual Start Date (only if C20 = Custom)  
- **C24** – End Date (`TODAY()` or manual if Custom)  
- **C25** – Price input used by the model (overwritten in backtest)
- **AA:AB** – `GOOGLEFINANCE` spill: **Date | Close**  
- **AC** – Returns from AB  
- **AD2:AD10** – Helpers (prev close, MAs, vol, 52-wk high/low)  
- **C5:C15** – Boolean rules (trend/vol/breakouts/gaps)  
- **E16** – Final decision (Buy / Sell / Hold)

**Design choices**
- Helpers derive from `AA:AB` to keep logic deterministic.  
- No volatile functions in the helper stack.  
- Guardrails keep helpers valid during warm-up.

### 2.2 Backtest sheet (evaluation workspace)
- **A:D** – Date | Ticker | Close | Action (logged by Apps Script)  
- **E:H** – Next Date | Next Close | Next Return | Correct?  
- **J1** – Horizon (steps ahead; weekly = next Friday when 1)  
- **J2** – Tolerance (dead-band, e.g., `0.001`)  
- **P1:P8** – Summary cells (total, correct, accuracy, baseline, p-value, counts)

---

## 3) Backtesting Engine (Apps Script)

- **Weekly endpoints (fast):** one row per trading week.  
- **Daily (optional):** every day; slower.

**Key details**
- Read spill once, iterate in memory.  
- Set `C24` to test date to prevent look-ahead.  
- Inject close into `C25`; read normalized `E16` (Buy/Sell/Hold).  
- Batch writes; small wait loop for spill readiness.  
- `onOpen()` menu for one-click run/clear.

**Why robust**
- Same logic as live (no duplicated rule code).  
- Weekly endpoints + batching avoid timeouts.  
- Explicit “as-of” date stops future leakage.

---

## 4) Evaluation Methodology

- **Horizon (J1):** steps ahead (1 = next row; weekly ≈ next week).  
- **Tolerance (J2):** dead-band so tiny moves are noise.  
- **Correct?**  
  - Buy correct if next return ≥ +tolerance (or ≥ 0 if strict=0)  
  - Sell correct if next return ≤ −tolerance  
  - Hold correct if not meaningfully negative (strict variant: within ±tolerance)
- **Baseline:** fraction where next return ≥ −tolerance (market drift).  
- **Accuracy/Edge/Significance:** accuracy − baseline; one-sided binomial p-value.

---

## 5) Results (general)
Model’s weekly next-bar calls achieved higher accuracy than baseline; improvement significant by a one-sided binomial test. Buy/Hold added value; Sell best as risk-reduction/exit. Figures are computed live in the dashboard.

### 5.1 Worked Example — AAPL (Weekly)
Setup: `J1=1`, `J2=0.0014`, range ≈ 4y.  
Core: 208 signals, accuracy 52.9% vs baseline 45.67%, p ≈ 0.0219.  
Interpretation: Outperforms naive baseline for next-week direction; prefer Sell=exit over shorting. Run quick robustness checks.

---

## 6) Operating the System
1. Choose ticker in `Dashboard!C19`.  
2. Select range in `C20` (Custom uses `C22/C26`).  
3. Run backtest from menu (Weekly recommended).  
4. Review `Backtest`: Next Date/Close/Return/Correct?  
5. Read summary: accuracy, baseline, p-value, proof line.  
6. Optional: daily snapshot trigger.

---

## 7) Risks, Assumptions & Mitigations
- `GOOGLEFINANCE` delays/throttling → spill readiness checks; restartable.  
- Warm-up periods → guards against short samples/div-by-zero.  
- Look-ahead bias → explicit as-of date before computing metrics.  
- Multiple testing/overfitting → simple rules, vary horizon/tolerance, OOS.  
- Transaction costs → add with simple formulas if needed.

---

## 8) Future Enhancements
Equity curve & drawdowns; strategy vs. benchmark chart; per-ticker roll-ups; parameter sweeps; CI bands; optional external data.

---

## Appendices
<font color="#0f172a">

<details>
<summary><strong>Appendix A — Key Formulas (copy-ready)</strong></summary>

#### A.1 Date range controls (Dashboard)
```text
C21 (Start Date; EDATE version with 4-year option)
=IF($C$20="YTD",
  DATE(YEAR(TODAY()),1,1),
  IF($C$20="Custom",
    $C$22,
    EDATE(TODAY(), -SWITCH($C$20,
      "1 Month",1,
      "3 Months",3,
      "6 Months",6,
      "1 Year",12,
      "2 Years",24,
      "3 Years",36,
      "4 Years",48,
      "5 Years",60,
      12
    ))
  )
)

C24 (End Date)
=IF($C$20="Custom", $C$26, TODAY())

C23 (Days in range)
=IF($C$20="YTD",
  TODAY()-DATE(YEAR(TODAY()),1,1),
  IF($C$20="Custom",
    MAX(0,$C$24-$C$22),
    $C$24 - $C$21
  )
)
````

#### A.2 History & returns (Dashboard)

```text
AA2 (history spill)
=GOOGLEFINANCE($C$19,"close",$C$21,$C$24,"DAILY")

AC3 (returns)
=ARRAYFORMULA(IF(ROW(AB3:AB)=ROW(AB3),"",IF(AB3:AB="","",AB3:AB/AB2:AB-1)))
```

#### A.3 Helper metrics (Dashboard!AD2:AD10; non-volatile, guarded)

```text
AD2 – count of dates
=COUNT($AA$3:$AA)

AD3 – previous close
=IF($AD$2>=1, INDEX($AB$3:$AB,$AD$2), )

AD4 – MA10
=IF($AD$2>=1, AVERAGE( INDEX($AB$3:$AB, MAX(1,$AD$2-9)) : INDEX($AB$3:$AB,$AD$2) ), )

AD5 – MA30
=IF($AD$2>=1, AVERAGE( INDEX($AB$3:$AB, MAX(1,$AD$2-29)) : INDEX($AB$3:$AB,$AD$2) ), )

AD6 – MA10 (yesterday)
=IF($AD$2>=2, AVERAGE( INDEX($AB$3:$AB, MAX(1,$AD$2-10)) : INDEX($AB$3:$AB,$AD$2-1) ), )

AD7 – MA10 (2 days ago)
=IF($AD$2>=3, AVERAGE( INDEX($AB$3:$AB, MAX(1,$AD$2-11)) : INDEX($AB$3:$AB,$AD$2-2) ), )

AD8 – Volatility % (annualized, last up to 30 returns)
=IF(COUNT($AC$3:$AC)<2, "",
 STDEV( OFFSET($AC$3, MAX(0, COUNT($AC$3:$AC)-30), 0, MIN(30, COUNT($AC$3:$AC)), 1) ) * SQRT(252) * 100
)

AD9 – 52-week high
=IF($AD$2>=1, MAX( INDEX($AB$3:$AB, MAX(1,$AD$2-251)) : INDEX($AB$3:$AB,$AD$2) ), )

AD10 – 52-week low
=IF($AD$2>=1, MIN( INDEX($AB$3:$AB, MAX(1,$AD$2-251)) : INDEX($AB$3:$AB,$AD$2) ), )
```

#### A.4 Rules & decision (Dashboard)

```text
// Examples for C5:C15 (adapt to your preferences)
C5  =AND($C$25>$AD$4,$C$25>$AD$5)                // above MA10 & MA30
C6  =AND($C$25<$AD$4,$C$25<$AD$5)                // below MA10 & MA30
C7  =$AD$8>30                                    // high vol flag
C8  =$C$25>$AD$9                                 // above 52-week high
C9  =$C$25<$AD$10                                // below 52-week low
C10 =AND($AD$8<=20, ABS($AD$4-$AD$5)/AVERAGE($AD$4,$AD$5) < 0.005)
C11 =AND($AD$4>$AD$6,$AD$6>$AD$7)                // MA10 rising
C12 =AND($AD$4<$AD$6,$AD$6<$AD$7)                // MA10 falling
C13 =AND($C$25>=MIN($AD$4,$AD$5), $C$25<=MAX($AD$4,$AD$5))
C14 =IFERROR($C$25/$AD$3-1>0.03,FALSE)           // gap up > 3%
C15 =IFERROR($C$25/$AD$3-1<-0.03,FALSE)          // gap down < 3%

E16 (decision)
=IFS(
  C12, "Sell",
  C6,  "Sell",
  C9,  "Sell",
  C11, "Buy",
  C5,  "Buy",
  C7,  "Hold",
  C13, "Hold",
  TRUE,"Hold"
)
```

#### A.5 Backtest derived columns (Backtest)

```text
// Settings: J1 = horizon (e.g., 1), J2 = tolerance (e.g., 0 or 0.001).

// Drag-friendly version
E2 =IF(INDEX($B:$B,ROW()+$J$1)<>B2,"",INDEX($A:$A,ROW()+$J$1))
F2 =IF(E2="","",INDEX($C:$C,ROW()+$J$1))
G2 =IF(F2="","",F2/C2-1)
H2 =IF(G2="","",IF(D2="Buy",N(G2>=$J$2),IF(D2="Sell",N(G2<=-$J$2),IF(D2="Hold",N(G2>=$J$2),""))))

// Strict Hold variant (optional)
H2 =IF(G2="","",
 IF(D2="Buy",  N(G2>$J$2),
 IF(D2="Sell", N(G2<$J$2),
 IF(D2="Hold", N(ABS(G2)<=$J$2),""))))
```

#### A.6 Summary cells & proof (Backtest)

```text
P1 =COUNT($H$2:$H)                                      // total signals
P2 =SUM($H$2:$H)                                         // correct signals
P3 =IFERROR($P$2/$P$1,"")                               // accuracy
P4 =IFERROR( COUNTIF($G$2:$G,">="&(-$J$2)) / COUNTIF($G$2:$G,"<>"), "")   // baseline
P5 =IFERROR( 1 - BINOM.DIST($P$2-1,$P$1,$P$4,TRUE), "") // one-sided p-value

One-line proof (e.g., R1)
=IF(OR($P$1=0,$P$4="",$P$3=""),"",
IF(AND($P$3>$P$4,$P$5<0.05),
"✅ Statistically significant improvement: "&Dashboard!C19&" ("&$J$1&"-step horizon, tol="&TEXT($J$2,"0.0%")&") achieved "&TEXT($P$3,"0.00%")&" accuracy across "&$P$1&" signals, beating baseline "&TEXT($P$4,"0.00%")&" (p="&TEXT($P$5,"0.0000")&").",
"⚠️ Not significant: accuracy "&TEXT($P$3,"0.00%")&" vs baseline "&TEXT($P$4,"0.00%")&" (p="&TEXT($P$5,"0.0000")&")."))
```

</details>

<details>
<summary><strong>Appendix B — Apps Script (compact skeleton)</strong></summary>

```text
function onOpen(){
  SpreadsheetApp.getUi()
    .createMenu('Backtest')
    .addItem('Run WEEKLY (1 ticker)','runBacktestWeeklyOneTicker')
    .addItem('Run DAILY (1 ticker)','runBacktestDailyOneTicker')
    .addItem('Clear Backtest','clearBacktest')
    .addToUi();
}

function runBacktestWeeklyOneTicker(){
  const ss=SpreadsheetApp.getActive(), dash=ss.getSheetByName('Dashboard');
  const out=ss.getSheetByName('Backtest')||ss.insertSheet('Backtest');
  if(out.getLastRow()===0) out.appendRow(['Date','Ticker','Close','Action']);

  const start=dash.getRange('C21').getValue();
  const end  =dash.getRange('C24').getValue();
  const c25=dash.getRange('C25');
  const origEnd=end, c25f=c25.getFormula(), c25v=c25.getValue();

  if(!waitForSpill_(dash,10000)) throw new Error('Spill AA:AB not ready');
  const table=dash.getRange('AA3:AB100000').getValues();
  const days=[];
  for(let i=0;i<table.length;i++){
    const d=table[i][0],px=table[i][1];
    if(!(d instanceof Date)) break;
    if(d<start||d>end||typeof px!=='number') continue;
    days.push([d,px]);
  }
  const weekly=pickWeeklyEndpoints_(days);

  const tkr=dash.getRange('C19').getValue();
  const rows=[];
  for(const [d,px] of weekly){
    dash.getRange('C24').setValue(d);
    c25.setValue(px);
    SpreadsheetApp.flush();
    const action=normalizeAction_(dash.getRange('E16').getValue());
    rows.push([d,tkr,px,action]);
  }
  if(rows.length){
    const lr=out.getLastRow();
    out.getRange(lr+1,1,rows.length,4).setValues(rows);
  }
  dash.getRange('C24').setValue(origEnd);
  if(c25f) c25.setFormula(c25f); else c25.setValue(c25v);
}

function runBacktestDailyOneTicker(){ /* identical shape; iterate all days */ }

function pickWeeklyEndpoints_(daysAsc){
  const out=[];
  let prev=null,last=null;
  for(const r of daysAsc){
    const d=r[0];
    const key=isoWeekKey_(d);
    if(key!==prev&&last) out.push(last);
    prev=key;
    last=r;
  }
  if(last) out.push(last);
  return out;
}

function isoWeekKey_(d){
  const x=new Date(d);
  x.setHours(0,0,0,0);
  x.setDate(x.getDate()+3-((x.getDay()+6)%7));
  const w1=new Date(x.getFullYear(),0,4);
  const week=1+Math.round(((x-w1)/86400000-3+((w1.getDay()+6)%7))/7);
  return x.getFullYear()+"-"+("0"+week).slice(-2);
}

function waitForSpill_(sh,ms){
  const t=Date.now();
  while(Date.now()-t<ms){
    SpreadsheetApp.flush();
    const d=sh.getRange('AA3').getValue(), p=sh.getRange('AB3').getValue();
    if(d instanceof Date && typeof p==='number') return true;
    Utilities.sleep(200);
  }
  return false;
}

function normalizeAction_(v){
  v=String(v||'').toLowerCase();
  if(v.includes('sell')) return 'Sell';
  if(v.includes('buy')) return 'Buy';
  return 'Hold';
}

function clearBacktest(){
  const ss=SpreadsheetApp.getActive(), out=ss.getSheetByName('Backtest');
  if(out) out.clearContents().appendRow(['Date','Ticker','Close','Action']);
}
```

</details>

</font>

---

## Closing

This dashboard gives you a single source of truth for signal logic, backtesting, and evaluation. It’s transparent, parameterized, fast enough to iterate in real time, and rigorous enough to support a defensible claim when results are genuinely better than baseline. The approach is intentionally modular so you can extend it—add P&L, drawdowns, or per-ticker roll-ups—without changing the core model logic.

```

That’s one clean block. It removes your `<style>` (GitHub ignores it), fixes the broken “text / Copy code” leftovers, closes each `<details>` properly, and keeps **Appendix B** + **Closing** outside **Appendix A**.
```

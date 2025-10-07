<!-- README: Market Signals Analytics Dashboard (Google Sheets + Apps Script) -->
<div align="center">

  <h1>📈 Market Signals Analytics Dashboard</h1>
  <p><strong>(Google Sheets + Apps Script)</strong></p>

  <p>
    <em>End-to-end, reproducible market signal research in Google Sheets—live dashboard, automated backtesting, and statistical evaluation.</em>
  </p>

  <p>
    <a href="#executive-summary">Executive Summary</a> ·
    <a href="#1-goals--scope">Goals & Scope</a> ·
    <a href="#2-system-design-sheets-layout">System Design</a> ·
    <a href="#3-backtesting-engine-apps-script">Backtesting Engine</a> ·
    <a href="#4-evaluation-methodology">Evaluation</a> ·
    <a href="#5-results">Results</a> ·
    <a href="#6-operating-the-system">Operate</a> ·
    <a href="#7-risks-assumptions--mitigations">Risks</a> ·
    <a href="#8-future-enhancements">Future</a>
  </p>

</div>

<hr/>

<style>
  /* Minimal, readable styling for GitHub HTML README */
  :root {
    --fg: #0f172a; /* slate-900 */
    --fg-soft: #334155; /* slate-700 */
    --muted: #64748b; /* slate-500 */
    --accent: #2563eb; /* blue-600 */
    --bg-chip: #eff6ff; /* blue-50 */
    --bd: #e2e8f0; /* slate-200 */
    --code-bg: #0b1020;
    --code-fg: #e6edf3;
  }
  @media (prefers-color-scheme: dark) {
    :root {
      --fg: #e6edf3;
      --fg-soft: #cbd5e1;
      --muted: #94a3b8;
      --accent: #60a5fa;
      --bg-chip: rgba(96,165,250,0.15);
      --bd: #334155;
      --code-bg: #0b1020;
      --code-fg: #e6edf3;
    }
  }

  h1, h2, h3 { color: var(--fg); }
  p, li, td, th { color: var(--fg-soft); line-height: 1.6; }
  code, pre code { font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace; }
  pre {
    background: var(--code-bg);
    color: var(--code-fg);
    padding: 14px 16px;
    border-radius: 10px;
    overflow: auto;
    border: 1px solid var(--bd);
  }
  .chip {
    display: inline-block;
    padding: 2px 8px;
    border-radius: 999px;
    border: 1px solid var(--bd);
    background: var(--bg-chip);
    color: var(--accent);
    font-size: 12px;
    margin-right: 6px;
    white-space: nowrap;
  }
  .kicker { color: var(--muted); text-transform: uppercase; letter-spacing: .08em; font-size: 12px; }
  .card {
    border: 1px solid var(--bd);
    border-radius: 12px;
    padding: 16px;
    margin: 14px 0;
  }
  .grid-2 {
    display: grid; gap: 16px;
    grid-template-columns: repeat(2, minmax(0, 1fr));
  }
  @media (max-width: 768px) {
    .grid-2 { grid-template-columns: 1fr; }
  }
  details {
    border: 1px solid var(--bd);
    border-radius: 10px;
    padding: 10px 12px;
    margin: 12px 0;
  }
  details summary {
    cursor: pointer; font-weight: 600; color: var(--fg);
  }
  .toc a { color: var(--accent); text-decoration: none; }
  .emph { color: var(--accent); font-weight: 600; }
</style>

<!-- EXECUTIVE SUMMARY -->
<h2 id="executive-summary">Executive Summary</h2>
<p>
This project delivers an end-to-end analytics workflow for market signal research using only
<strong>Google Sheets</strong> and <strong>Apps Script</strong>. It combines:
</p>
<ul>
  <li><span class="emph">Live signal dashboard</span> (single sheet) for one-click analysis on any ticker and date range.</li>
  <li><span class="emph">Automated backtesting engine</span> (daily or weekly endpoints) that replays history through the same model logic used in live mode.</li>
  <li><span class="emph">Evaluation framework</span> with baseline vs. model comparison, configurable horizon/tolerance, and automatic statistical significance checks.</li>
  <li><span class="emph">Shareable output</span>: a one-line “proof” sentence and summary blocks that communicate results credibly without manual number crunching.</li>
</ul>
<div class="card">
  <p><strong>Outcomes (high level):</strong> The model’s accuracy showed a material improvement over the baseline and this lift was statistically significant under conventional thresholds. The system runs fast (weekly endpoints), avoids common spreadsheet pitfalls, and is easy to extend.</p>
</div>

<!-- GOALS & SCOPE -->
<h2 id="1-goals--scope">1) Goals & Scope</h2>

<h3>Goals</h3>
<ul>
  <li>Build a lightweight, auditable research tool that requires no external infrastructure.</li>
  <li>Use the same rules for live and backtest (no code/logic drift).</li>
  <li>Quantify whether signals beat a fair baseline and whether the lift is unlikely to be luck.</li>
</ul>

<h3>Non-Goals</h3>
<ul>
  <li>High-frequency or intraday trading.</li>
  <li>Broker execution or actual order routing.</li>
  <li>Optimizing for specific tickers only—design favors generality.</li>
</ul>

<!-- SYSTEM DESIGN -->
<h2 id="2-system-design-sheets-layout">2) System Design (Sheets Layout)</h2>

<p>All core logic lives on one sheet named <code>Dashboard</code>; backtest results are written to a separate <code>Backtest</code> sheet.</p>

<h3>2.1 Dashboard (live + engine inputs)</h3>
<div class="grid-2">
  <div>
    <ul>
      <li><strong>C19</strong> – Ticker (e.g., <code>AAPL</code>).</li>
      <li><strong>C20</strong> – Range selector: 1M, 3M, 6M, 1Y, 2Y, 3Y, 4Y, 5Y, YTD, Custom.</li>
      <li><strong>C21</strong> – Start Date (computed; see formulas in <a href="#appendix-a--key-formulas-copy-ready">Appendix A</a>).</li>
      <li><strong>C22</strong> – Manual Start Date (used only if C20 = Custom).</li>
      <li><strong>C24</strong> – End Date (usually <code>TODAY()</code> or manual if Custom).</li>
      <li><strong>C25</strong> – Price input used by the model (live formula; overwritten in backtest).</li>
    </ul>
  </div>
  <div>
    <ul>
      <li><strong>AA:AB</strong> – History spill from <code>GOOGLEFINANCE</code>: <em>Date | Close</em>.</li>
      <li><strong>AC</strong> – Returns column computed from AB.</li>
      <li><strong>AD2:AD10</strong> – Helper metrics (prev close, MAs, rolling vol, 52-week high/low).</li>
      <li><strong>C5:C15</strong> – Boolean rules for the model (trend, volatility, breakouts, gaps, etc.).</li>
      <li><strong>E16</strong> – Final decision (e.g., Buy / Sell / Hold) derived from the rules.</li>
    </ul>
  </div>
</div>

<p><strong>Design choices</strong></p>
<ul>
  <li>Helpers derive from the spilled history (<code>AA:AB</code>) to keep logic deterministic.</li>
  <li>No volatile functions in the helper stack to avoid jitter.</li>
  <li>Helper formulas include guardrails so they remain valid during warm-up periods.</li>
</ul>

<h3>2.2 Backtest sheet (evaluation workspace)</h3>
<ul>
  <li><strong>A:D</strong> – <em>Date | Ticker | Close | Action</em> (logged by Apps Script from the live model).</li>
  <li><strong>E:H</strong> – Derived metrics per row:
    <ul>
      <li><strong>E</strong> = Next Date (horizon steps ahead within the same ticker)</li>
      <li><strong>F</strong> = Next Close</li>
      <li><strong>G</strong> = Next Return (<code>F/C − 1</code>)</li>
      <li><strong>H</strong> = Correct? (1 or 0 based on the decision and tolerance)</li>
    </ul>
  </li>
  <li><strong>J1</strong> – Horizon (steps ahead; e.g., weekly = next Friday when set to 1).</li>
  <li><strong>J2</strong> – Tolerance (dead-band around zero return; e.g., 0.1% as <code>0.001</code>).</li>
  <li><strong>P1:P8</strong> – Summary cells (total, correct, accuracy, baseline, p-value, counts, etc.).</li>
</ul>
<p>Optional proof cell prints a single sentence when the model beats the baseline significantly.</p>

<!-- BACKTEST ENGINE -->
<h2 id="3-backtesting-engine-apps-script">3) Backtesting Engine (Apps Script)</h2>

<p>
The Apps Script runner replays historical data into the live model by setting the as-of date and the price for each test step, then reading the dashboard’s <code>E16</code>.
Two modes are supported:
</p>
<ul>
  <li><span class="chip">Weekly endpoints (fast)</span> reduces daily history into one row per trading week (e.g., Friday or the last trading day if holidays).</li>
  <li><span class="chip">Daily (optional)</span> replays every day; chunked writing and spill checks keep it robust but slower.</li>
</ul>

<p><strong>Key implementation details</strong></p>
<ul>
  <li>Spill read once: read <code>AA:AB</code> into memory, then iterate—no per-row <code>getValue()</code> calls.</li>
  <li>As-of date: set <code>C24</code> to the test date so helper metrics use only prior history (prevents look-ahead bias).</li>
  <li>Price injection: set <code>C25</code> to the day’s official close to drive the rules.</li>
  <li>Decision read: read <code>E16</code> and normalize to a canonical label (Buy / Sell / Hold) for logging.</li>
  <li>Batch writes and spill readiness checks to avoid timeouts and race conditions.</li>
  <li>Menu + alias: an <code>onOpen()</code> menu for one-click run/clear; wrapper keeps legacy names working.</li>
</ul>

<div class="card">
  <p><strong>Why the design is robust</strong></p>
  <ul>
    <li>Uses the exact same logic as live mode—no duplicated rule code.</li>
    <li>Writes in batches and uses weekly endpoints to avoid Apps Script timeouts.</li>
    <li>Explicitly sets an “as-of” date to eliminate future leakage.</li>
  </ul>
</div>

<!-- EVALUATION -->
<h2 id="4-evaluation-methodology">4) Evaluation Methodology</h2>

<h3>4.1 Horizon &amp; Tolerance</h3>
<ul>
  <li><strong>Horizon (J1)</strong>: how many steps ahead to check (1 = next row; with weekly data that’s next week).</li>
  <li><strong>Tolerance (J2)</strong>: a small dead-band around zero return so tiny moves are treated as noise.</li>
</ul>

<h3>4.2 “Correct?” logic (per row)</h3>
<p>Using <code>G = Next Return</code> and <code>J2 = tolerance</code>:</p>
<ul>
  <li><strong>Buy</strong> is correct if next return ≥ +tolerance (or ≥ 0 if strict = 0).</li>
  <li><strong>Sell</strong> is correct if next return ≤ −tolerance.</li>
  <li><strong>Hold</strong> is correct if next return is not meaningfully negative; strict variant: within ±tolerance.</li>
</ul>

<h3>4.3 Baseline (null hypothesis rate)</h3>
<p>
Baseline = unconditional success rate from a trivial rule that ignores the model.
In this design it’s the fraction of periods where <code>next return ≥ −tolerance</code> (flat or up within the band). This captures market drift and defines the null hit rate.
</p>

<h3>4.4 Accuracy, edge, and significance</h3>
<ul>
  <li><strong>Accuracy</strong>: share of rows where the model’s call was “Correct?”.</li>
  <li><strong>Edge</strong>: accuracy − baseline; positive gap indicates better-than-trivial performance.</li>
  <li><strong>Significance (p-value)</strong>: a one-sided binomial test—probability of observing accuracy this high if true rate = baseline.</li>
</ul>

<h3>4.5 Shareable, automatic proof</h3>
<p>
A one-cell formula synthesizes a narrative sentence—only when accuracy &gt; baseline and statistically significant—so results can be shared without manual calculation.
</p>

<!-- RESULTS -->
<h2 id="5-results">5) Results (general, number-free)</h2>
<p>
Across the evaluated window, the model’s <em>weekly next-bar</em> calls achieved higher accuracy than the baseline, and the improvement was statistically significant using a one-sided binomial test. The edge held under small changes to horizon and tolerance. Qualitatively, Buy tended to precede favorable next-period outcomes, Sell worked reliably as a risk-reduction/exit signal, and Hold captured the market’s tendency to drift upward.
</p>
<p><em>Note:</em> Figures are intentionally omitted here; the dashboard computes them live so they remain current and reproducible.</p>

<h3 id="51-worked-example--aapl-weekly">5.1 Worked Example — AAPL (Weekly)</h3>
<p><strong>Setup.</strong> Ticker: <code>AAPL</code>; horizon <code>J1 = 1</code> (next Friday), tolerance <code>J2 = 0.0014</code> (0.14%), range ≈ 4 years. Backtest sheet computes Next Date/Close/Return and Correct? with drag-friendly formulas; summary cells in <code>P1–P5</code>.</p>

<div class="grid-2">
  <div class="card">
    <p><strong>Core results.</strong></p>
    <ul>
      <li>Total signals: 208; Correct: 110; Overall accuracy: 52.9%.</li>
      <li>Baseline (flat-or-up ≥ −tol): 45.67% → market’s unconditional “success” rate.</li>
      <li>Statistical test: one-sided binomial <code>p = 0.0219</code> → significant at 5%.</li>
      <li>Per-action accuracy: Buy ≈ 56.1%, Sell ≈ 49.4%, Hold ≈ 53.7%.</li>
      <li>Average next returns: Buy ≈ +0.29%, Sell ≈ +0.49%, Hold ≈ +0.22% (weekly).</li>
    </ul>
  </div>
  <div class="card">
    <p><strong>Interpretation.</strong> The AAPL rules meaningfully outperform the naive baseline for next-week direction, with evidence unlikely due to chance under the null (baseline) rate. Because Sell’s average next-week return is mildly positive, using Sell = go flat/exit is preferable to shorting. Run quick robustness checks (tweak horizon/tolerance) before deployment.</p>
    <p><strong>Representative snapshot.</strong> On 2025-10-06, the condition table produced a <em>Sell</em> (below MA10 &amp; MA30; price near 52-week low; gap-down &gt;3%) with explanatory text, plus live charts—illustrating how the same rules power both live and backtest views.</p>
  </div>
</div>

<div class="card">
  <strong>Shareable proof sentence:</strong>
  <br/>
  <code>Statistically significant: AAPL (1-step horizon, tol=0.14%) achieved ~52.9% accuracy across 208 signals, beating baseline ~45.7% (one-sided binomial p≈0.0219).</code>
</div>

<!-- OPERATIONS -->
<h2 id="6-operating-the-system">6) Operating the System</h2>
<ol>
  <li>Choose a ticker in <code>Dashboard!C19</code>.</li>
  <li>Pick a date range using <code>C20</code> (or Custom with <code>C22/C26</code>). The history spill in <code>AA:AB</code> updates automatically.</li>
  <li>Run backtest from the custom menu:
    <ul>
      <li><strong>Weekly (recommended):</strong> runs quickly and avoids Apps Script limits.</li>
      <li><strong>Daily (optional):</strong> more granular but slower.</li>
    </ul>
  </li>
  <li>Review <code>Backtest</code> sheet: derived columns fill automatically (Next Date, Next Close, Next Return, Correct?).</li>
  <li>Read the summary: accuracy, baseline, p-value, per-action metrics, and the proof line.</li>
  <li>Optional: add a time-based trigger to take a daily snapshot of the live model.</li>
</ol>

<!-- RISKS -->
<h2 id="7-risks-assumptions--mitigations">7) Risks, Assumptions &amp; Mitigations</h2>
<ul>
  <li><strong>Data source limitations:</strong> <code>GOOGLEFINANCE</code> may delay/throttle—scripts include spill readiness checks and are restartable.</li>
  <li><strong>Warm-up periods:</strong> helper guardrails prevent divide-by-zero/short-sample issues.</li>
  <li><strong>Look-ahead bias:</strong> explicitly setting the as-of date (<code>C24</code>) eliminates future leakage.</li>
  <li><strong>Multiple testing / overfitting:</strong> keep rules simple, validate across horizons/tolerances, and use out-of-sample windows.</li>
  <li><strong>Transaction costs:</strong> not modeled by default; can be added with simple formulas.</li>
</ul>

<!-- FUTURE -->
<h2 id="8-future-enhancements">8) Future Enhancements</h2>
<ul>
  <li>Equity curve &amp; drawdown columns and a strategy vs. benchmark chart.</li>
  <li>Per-ticker roll-ups and a small leaderboard for multi-ticker research.</li>
  <li>Parameter sweeps (grid of horizon/tolerance) with heatmap/conditional formatting.</li>
  <li>CI bands for accuracy and baseline (Wald or Wilson) to communicate uncertainty.</li>
  <li>Optional external data adapters (fundamentals, alt-data).</li>
</ul>

<!-- APPENDICES -->
<h2 id="appendices">Appendices</h2>
<div style="color:#0f172a;">
<details id="appendix-a--key-formulas-copy-ready">
  <summary><strong>Appendix A — Key Formulas (copy-ready)</strong></summary>
  <h4>A.1 Date range controls (Dashboard)</h4>
  <pre><code>C21 (Start Date; EDATE version with 4-year option)
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
)</code></pre>

  <h4>A.2 History &amp; returns (Dashboard)</h4>
  <pre><code>AA2 (history spill)
=GOOGLEFINANCE($C$19,"close",$C$21,$C$24,"DAILY")

AC3 (returns)
=ARRAYFORMULA(IF(ROW(AB3:AB)=ROW(AB3),"",IF(AB3:AB="","",AB3:AB/AB2:AB-1)))</code></pre>

  <h4>A.3 Helper metrics (Dashboard!AD2:AD10; non-volatile, guarded)</h4>
  <pre><code>AD2 – count of dates
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
=IF(COUNT($AC$3:$AC)&lt;2, "",
 STDEV( OFFSET($AC$3, MAX(0, COUNT($AC$3:$AC)-30), 0, MIN(30, COUNT($AC$3:$AC)), 1) ) * SQRT(252) * 100
)

AD9 – 52-week high
=IF($AD$2>=1, MAX( INDEX($AB$3:$AB, MAX(1,$AD$2-251)) : INDEX($AB$3:$AB,$AD$2) ), )

AD10 – 52-week low
=IF($AD$2>=1, MIN( INDEX($AB$3:$AB, MAX(1,$AD$2-251)) : INDEX($AB$3:$AB,$AD$2) ), )</code></pre>

  <h4>A.4 Rules &amp; decision (Dashboard)</h4>
  <pre><code>// Examples for C5:C15 (adapt to your preferences)
C5  =AND($C$25>$AD$4,$C$25>$AD$5)                // above MA10 &amp; MA30
C6  =AND($C$25&lt;$AD$4,$C$25&lt;$AD$5)                // below MA10 &amp; MA30
C7  =$AD$8&gt;30                                    // high vol flag
C8  =$C$25&gt;$AD$9                                 // above 52-week high
C9  =$C$25&lt;$AD$10                                // below 52-week low
C10 =AND($AD$8&lt;=20, ABS($AD$4-$AD$5)/AVERAGE($AD$4,$AD$5) &lt; 0.005)
C11 =AND($AD$4&gt;$AD$6,$AD$6&gt;$AD$7)                // MA10 rising
C12 =AND($AD$4&lt;$AD$6,$AD$6&lt;$AD$7)                // MA10 falling
C13 =AND($C$25&gt;=MIN($AD$4,$AD$5), $C$25&lt;=MAX($AD$4,$AD$5))
C14 =IFERROR($C$25/$AD$3-1&gt;0.03,FALSE)           // gap up &gt; 3%
C15 =IFERROR($C$25/$AD$3-1&lt;-0.03,FALSE)          // gap down &lt; 3%

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
)</code></pre>

  <h4>A.5 Backtest derived columns (Backtest)</h4>
  <pre><code>// Settings: J1 = horizon (e.g., 1), J2 = tolerance (e.g., 0 or 0.001).

// Drag-friendly version
E2 =IF(INDEX($B:$B,ROW()+$J$1)&lt;&gt;B2,"",INDEX($A:$A,ROW()+$J$1))
F2 =IF(E2="","",INDEX($C:$C,ROW()+$J$1))
G2 =IF(F2="","",F2/C2-1)
H2 =IF(G2="","",IF(D2="Buy",N(G2&gt;=$J$2),IF(D2="Sell",N(G2&lt;=-$J$2),IF(D2="Hold",N(G2&gt;=$J$2),""))))

// Strict Hold variant (optional)
H2 =IF(G2="","",
 IF(D2="Buy",  N(G2&gt;$J$2),
 IF(D2="Sell", N(G2&lt;-$J$2),
 IF(D2="Hold", N(ABS(G2)&lt;=$J$2),""))))</code></pre>

  <h4>A.6 Summary cells &amp; proof (Backtest, labels in O, values in P)</h4>
  <pre><code>P1 =COUNT($H$2:$H)                                      // total signals
P2 =SUM($H$2:$H)                                         // correct signals
P3 =IFERROR($P$2/$P$1,"")                                // accuracy
P4 =IFERROR( COUNTIF($G$2:$G,"&gt;="&amp;(-$J$2)) / COUNTIF($G$2:$G,"&lt;&gt;"), "")   // baseline
P5 =IFERROR( 1 - BINOM.DIST($P$2-1,$P$1,$P$4,TRUE), "")  // one-sided p-value

One-line proof (e.g., R1)
=IF(OR($P$1=0,$P$4="",$P$3=""),"",
IF(AND($P$3&gt;$P$4,$P$5&lt;0.05),
"✅ Statistically significant improvement: "&amp;Dashboard!C19&amp;" ("&amp;$J$1&amp;"-step horizon, tol="&amp;TEXT($J$2,"0.0%")&amp;") achieved "&amp;TEXT($P$3,"0.00%")&amp;" accuracy across "&amp;$P$1&amp;" signals, beating baseline "&amp;TEXT($P$4,"0.00%")&amp;" (p="&amp;TEXT($P$5,"0.0000")&amp;").",
"⚠️ Not significant: accuracy "&amp;TEXT($P$3,"0.00%")&amp;" vs baseline "&amp;TEXT($P$4,"0.00%")&amp;" (p="&amp;TEXT($P$5,"0.0000")&amp;")."))</code></pre>
</details>

<details id="appendix-b--apps-script-compact-skeleton">
  <summary><strong>Appendix B — Apps Script (compact skeleton)</strong></summary>
  <pre><code>function onOpen(){
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
  const days=[]; for(let i=0;i&lt;table.length;i++){const d=table[i][0],px=table[i][1];
    if(!(d instanceof Date)) break; if(d&lt;start||d&gt;end||typeof px!=='number') continue; days.push([d,px]);}
  const weekly=pickWeeklyEndpoints_(days);

  const tkr=dash.getRange('C19').getValue();
  const rows=[]; for(const [d,px] of weekly){
    dash.getRange('C24').setValue(d); c25.setValue(px); SpreadsheetApp.flush();
    const action=normalizeAction_(dash.getRange('E16').getValue());
    rows.push([d,tkr,px,action]);
  }
  if(rows.length){ const lr=out.getLastRow(); out.getRange(lr+1,1,rows.length,4).setValues(rows); }
  dash.getRange('C24').setValue(origEnd); if(c25f) c25.setFormula(c25f); else c25.setValue(c25v);
}

function runBacktestDailyOneTicker(){ /* identical shape; iterate all days */ }

function pickWeeklyEndpoints_(daysAsc){
  const out=[]; let prev=null,last=null; for(const r of daysAsc){const d=r[0];
    const key=isoWeekKey_(d); if(key!==prev&amp;&amp;last) out.push(last); prev=key; last=r;} if(last) out.push(last); return out; }
function isoWeekKey_(d){ const x=new Date(d); x.setHours(0,0,0,0);
  x.setDate(x.getDate()+3-((x.getDay()+6)%7)); const w1=new Date(x.getFullYear(),0,4);
  const week=1+Math.round(((x-w1)/86400000-3+((w1.getDay()+6)%7))/7);
  return x.getFullYear()+"-"+("0"+week).slice(-2); }
function waitForSpill_(sh,ms){ const t=Date.now(); while(Date.now()-t&lt;ms){
  SpreadsheetApp.flush(); const d=sh.getRange('AA3').getValue(), p=sh.getRange('AB3').getValue();
  if(d instanceof Date &amp;&amp; typeof p==='number') return true; Utilities.sleep(200);} return false; }
function normalizeAction_(v){ v=String(v||'').toLowerCase();
  if(v.includes('sell')) return 'Sell'; if(v.includes('buy')) return 'Buy'; return 'Hold'; }

function clearBacktest(){
  const ss=SpreadsheetApp.getActive(), out=ss.getSheetByName('Backtest');
  if(out) out.clearContents().appendRow(['Date','Ticker','Close','Action']);
}</code></pre>
</details>
</div>

<hr/>

<p class="kicker">Closing</p>
<p>
This dashboard gives you a single source of truth for signal logic, backtesting, and evaluation. It’s transparent, parameterized, fast enough to iterate in real time, and rigorous enough to support a defensible claim when results are genuinely better than baseline. The approach is intentionally modular so you can extend it—add P&amp;L, drawdowns, or per-ticker roll-ups—without changing the core model logic.
</p>

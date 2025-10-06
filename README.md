# 🧠 Market Signals Analytics Dashboard  
**Google Sheets + Apps Script | Real-Time Signal Generation, Backtesting & Automation**

![Project Banner](https://github.com/yourusername/yourrepo/assets/banner.png)  
*(Add your own banner or chart image here to make the page more engaging!)*

---

## 🚀 Overview  
The **Market Signals Analytics Dashboard** is a fully automated **Google Sheets + Apps Script** project for **time-series analysis**, **signal generation**, and **statistical backtesting**.  

It analyzes stock behavior using **live market data** from `GOOGLEFINANCE` and Yahoo Finance APIs — evaluating price actions, volatility, and moving averages to produce actionable **Buy / Sell / Hold** decisions.  

Everything runs inside Google Sheets: fast, transparent, reproducible, and no external APIs required.

---

## ⚙️ Core Features  

### 📊 Signal Generation  
- Custom rule-based model powered by live price data.  
- Inputs: **Stock Symbol**, **Date Range**, **Horizon**, and **Tolerance**.  
- Evaluates key market conditions such as:  
  - Price above/below MA10 or MA30  
  - Volatility > 30%  
  - Price breaking 52-week high/low  
  - Strong bullish/bearish MA10 trend (3+ days)  
  - Gap up/down > 3%  
  - MA10 ≈ MA30 convergence  
- Generates interpretable Buy / Sell / Hold recommendations with reasoning.

### 🔁 Automated Backtesting  
- Built a **single-sheet backtester** that replays daily or weekly historical closes through model rules.  
- Logs results in a structured format:  
Date | Ticker | Close | Action | Horizon | Tolerance | Accuracy

markdown
Copy code
- Evaluates hit rates, counts, averages, and one-sided binomial p-values to test statistical significance.  

### 📈 Evaluation Framework  
- Compares **model accuracy vs. baseline** under identical conditions.  
- Automatically computes statistical significance (p-value).  
- Example performance:  
- **Model:** 58.65%  
- **Baseline:** 47.24%  
- **p ≈ 0.0006 → Statistically Significant Edge**  
- Includes a “proof” generator for shareable result statements and clean summary blocks.  

### ⚡ Automation & Scalability  
- **Apps Script** automation for daily/weekly replays and scheduled snapshots.  
- Optimized with batched writes and lightweight formula design (no volatile functions).  
- Modular, reproducible, and designed for multi-ticker scalability.  

---

## 📐 Architecture Overview  

**Data Flow**

**Core Components**

| Module | Description |
|---------|-------------|
| **Data Loader** | Fetches OHLC, volume, volatility, RSI, and moving averages |
| **Rules Engine** | Evaluates 10+ custom logic checks for signal generation |
| **Backtester** | Replays historical data to validate performance |
| **Evaluator** | Calculates accuracy, baseline comparison, and p-values |
| **Automation Layer** | Runs scheduled updates, cleans logs, and snapshots results |

---

## 📊 Example Results  

| Metric | Model | Baseline | Significance |
|--------|--------|-----------|---------------|
| Accuracy | **58.65%** | 47.24% | p ≈ 0.0006 ✅ |
| Weekly Signals | 208 | 208 | — |
| Edge | +11.41% | — | Statistically Significant |

> 📜 *“Model demonstrates a statistically significant predictive edge at the 95% confidence level.”*

---

## 🧩 Tech Stack  

**Languages & Tools**
- Google Sheets  
- Google Apps Script  
- `GOOGLEFINANCE` API  
- Statistical Testing (Binomial Distribution)  
- Dashboard Automation & ETL  

**Buzzwords:**  
`Google Apps Script · Google Sheets · GOOGLEFINANCE · Time-Series Analysis · Backtesting · Statistical Testing · Automation · Dashboards · ETL · Reproducibility`

---

## 💡 Learnings & Impact  
- Built a **no-code + low-script** analytical system entirely within Google Sheets.  
- Achieved a **statistically significant model** outperforming baseline results.  
- Delivered a **transparent**, **reproducible**, and **automated** analytics framework.  
- Created a flexible foundation for multi-ticker or multi-market expansion.  

---

## 🧠 How to Use  

1. **Make a Copy** of the main Google Sheet (link in repo).  
2. Enter a stock symbol (e.g., `AAPL`, `GOOG`, `TSLA`).  
3. Adjust **Horizon** and **Tolerance** settings.  
4. Run Apps Script functions like `Backtest()` or `ReplayWeekly()`.  
5. Review the dashboard for accuracy, p-value, and “proof” statement summaries.  

---

## 🧾 Example Rules Used  

| Rule | Condition | Decision Impact |
|------|------------|-----------------|
| Price > MA10 & MA30 | Strong momentum | Buy |
| Price < MA10 & MA30 | Downtrend | Sell |
| Volatility > 30% | Unstable | Hold |
| Price > 52-Week High | Breakout | Buy |
| Price < 52-Week Low | Weakness | Sell |
| Gap Up > 3% | Positive sentiment | Buy |
| Gap Down > 3% | Negative sentiment | Sell |
| MA10 rising 3+ days | Bullish trend | Buy |
| MA10 falling 3+ days | Bearish trend | Sell |

---

## 🧮 Example Data Pulled  

| Column | Description |
|---------|-------------|
| Date | Daily or weekly observation |
| Open / High / Low / Close | Standard OHLC data |
| Volume | Daily traded volume |
| Daily Return | Calculated % change |
| MA10 / MA30 | Moving averages |
| Volatility (20-day) | Standard deviation of returns |
| RSI (14-day) | Relative strength indicator |

---

## 📬 Contact  

**👤 Fadi Batshon**  
Business Economics & Finance @ UC San Diego  
📧 [Your Email Here]  
🌐 [Your LinkedIn or Website]  

---

⭐ *If you find this project useful, consider giving it a star on GitHub!*  

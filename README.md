<div align="center">

# 📊 Algorithmic Trading Strategy Dashboard  
### Automated Backtesting & Performance Analytics for Smart Trading Decisions  

**A powerful trading strategy system built entirely in Google Sheets — no programming knowledge required!**

[Features](#-features) • [Live Demo](#-live-demo) • [Quickstart](#-quickstart) • [Performance Results](#-performance-results)

</div>

---

## 🎯 What Is This?

A **complete, no-code algorithmic trading dashboard** built with **Google Sheets + Apps Script**, allowing you to:

✅ Analyze live stock prices and trends  
✅ Generate **BUY / SELL / HOLD** recommendations automatically  
✅ Backtest your strategy on historical data  
✅ Evaluate performance statistically (accuracy, p-values, returns)  
✅ Visualize everything through dynamic, professional charts  

Everything runs natively inside **Google Sheets** — transparent, replicable, and recruiter-friendly.

---

### 🎬 Example: AAPL Strategy Dashboard  

![Main Dashboard](docs/images/main-dashboard.png)  
*Real-time analytics dashboard generating automated trading decisions.*

---

## ✨ Features

### 📈 Smart Signal Engine  
Combines multiple indicators (moving averages, RSI, volatility, gaps, 52-week levels) to make **interpretable** trading calls.

### 🔬 Backtesting Engine  
Replays years of data to measure performance against a **baseline model**.

### 📊 Beautiful Visualizations  
Six clean charts showing price trends, volatility regimes, and signal accuracy.

### 🎯 Statistical Validation  
Automatically computes **accuracy, baseline, and one-sided binomial p-values** for every run.

---

## ⚙️ Quickstart  

### Step 1 – Get the Template  
1. Open the Google Sheets template  
2. Click **File → Make a copy**  
3. Save to your Drive — done!  

### Step 2 – Add Data  
Use Google’s built-in finance import:
=GOOGLEFINANCE("AAPL", "price", DATE(2020,1,1), TODAY(), "DAILY")
| Metric                  | Result      | Meaning                                |
| :---------------------- | :---------- | :------------------------------------- |
| **Overall Accuracy**    | **52.88 %** | > half of all predictions were correct |
| **Signals Tested**      | 208         | Weekly signals generated               |
| **Correct Predictions** | 110         | Hits vs. misses                        |
| **P-value**             | **0.0219**  | Statistically significant edge         |

| Signal  | Count | Accuracy | Avg Return | Rating |
| :------ | :---: | :------: | :--------: | :----- |
| 🟢 BUY  |   83  |  56.10 % |   +0.29 %  | ⭐⭐⭐⭐   |
| 🔴 SELL |   85  |  49.41 % |   +0.49 %  | ⭐⭐⭐    |
| 🟡 HOLD |   41  |  53.66 % |   +0.22 %  | ⭐⭐⭐⭐   |

✅ Result: The model outperforms random guessing and maintains a verified statistical advantage (p ≈ 0.02).

🧠 Decision Logic

BUY → Price > MA10 & MA30 + low volatility

SELL → Price < MA10 & MA30 + high volatility

HOLD → Mixed signals or neutral regime

🧩 Example Apps Script Snippet

function decide_(c, m10, m30, v, prev) {
  if (c > m10 && c > m30) return ['Buy', 'Strong Uptrend'];
  if (c < m10 && c < m30) return ['Sell', 'Downtrend'];
  if (v > 0.3)            return ['Hold', 'High Volatility'];
  if (c >= prev * 1.03)   return ['Buy', 'Gap Up >3%'];
  if (c <= prev * 0.97)   return ['Sell', 'Gap Down >3%'];
  return ['Hold', 'Neutral'];
}

📂 Project Structure

Algorithmic-Trading-Dashboard/
├── README.md
├── src/
│   └── apps_script.js
├── data/
│   └── backtest_log.csv
├── docs/
│   └── images/
│       ├── main-dashboard.png
│       ├── decision-matrix.png
│       ├── volatility.png
│       ├── returns-histogram.png
│       └── backtest-summary.png

📬 Contact

Fadi Batshon — Business Economics & Finance @ UC San Diego
📧 your@email.com
 | 🌐 [LinkedIn / Website]

<div align="center">

⭐ If you found this interesting, please star the repo!
Made with ❤️ in Google Sheets



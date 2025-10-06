
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

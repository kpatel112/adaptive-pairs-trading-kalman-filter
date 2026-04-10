# 📈 Statistical Arbitrage with Kalman Filter: Adaptive Pairs Trading

## 📌 Overview

This project implements a **statistical arbitrage (pairs trading)** strategy comparing:

* **Static hedge ratios** using Ordinary Least Squares (OLS)
* **Dynamic hedge ratios** using a Kalman Filter

The objective is to evaluate whether **time-varying relationships between assets** can improve trading performance over traditional static models.

---

## 🧠 Key Idea

Traditional pairs trading assumes a **constant relationship** between two assets.
This project relaxes that assumption by allowing the hedge ratio (β) to evolve over time using a **state-space model (Kalman Filter)**.

---

## 📊 Data

* **Universe:** 16 North American banking stocks
  (e.g., BAC, JPM, WFC, GS, MS, PNC, etc.)
* **Frequency:** Daily
* **Period:** 2021 – 2026
* **Source:** Yahoo Finance (`yfinance`)
* **Price Type:** Adjusted Close

### Preprocessing

* Removed tickers with >5% missing data
* Forward-filled small gaps (1–2 days)
* **Converted prices to log scale** for:

  * variance stabilization
  * improved linear relationships
  * more reliable cointegration testing

---

## 🔍 Methodology

### 1. Train-Test Split

* **Train:** First ~3 years
* **Test:** Last ~2 years
* Ensures **no lookahead bias**

---

### 2. Pair Selection (Cointegration)

* Tested all stock pairs using **Engle-Granger test**
* Regression:
  [
  P1_t = \alpha + \beta P2_t + \epsilon_t
  ]
* Applied **ADF test on residuals**
* Selected pairs with:

  * p-value < 0.05
  * Half-life between **5–30 days**

✅ Result:
Only **1 robust pair (BAC, PNC)** passed all filters after log transformation
→ indicates removal of false positives

---

### 3. OLS Model (Baseline)

* Estimated **static hedge ratio (β)**
* Constructed spread:
  [
  Spread_t = P1_t - \beta P2_t
  ]

---

### 4. Kalman Filter Model (Core Contribution)

State-space formulation:

* **State:**
  [
  x_t = [\beta_t, \alpha_t]
  ]

* **Observation:**
  [
  P1_t = \beta_t P2_t + \alpha_t + \epsilon_t
  ]

* Parameters:

  * Observation noise (R): variance of price changes
  * Process noise (Q): controls β adaptability

* Tested:
  [
  Q \in {10^{-6}, 10^{-5}, 10^{-4}, 10^{-3}}
  ]

* Selected best Q based on **smoothness of β(t)**

---

### 5. Signal Generation

* Rolling window: **60 days**
* Z-score:
  [
  z_t = \frac{Spread_t - \mu_t}{\sigma_t}
  ]

### Trading Rules

* **Long entry:** ( z < -2 )
* **Short entry:** ( z > +2 )
* **Exit:** ( |z| < 0.5 )
* **Stop-loss:** ( |z| > 3 )

---

### 6. Backtesting

* Conducted on **test data only**
* Strategy:

  * Market-neutral long/short positions
  * Daily signal evaluation
* Included **transaction costs: 0.1% per trade**

---

## 📈 Results

| Pair | OLS Sharpe | OLS Sortino | OLS Return | Kalman Sharpe | Kalman Sortino | Kalman Return |
|------|-----------|------------|-----------|--------------|---------------|--------------|
| BAC–PNC | -0.17 | -0.18 | -0.043 | **0.39** | **0.30** | **6.97** |

---

## 🧠 Key Findings

- The **OLS model failed** to generate positive risk-adjusted returns (negative Sharpe and Sortino).
- The **Kalman Filter model outperformed OLS**, achieving:
  - Positive Sharpe and Sortino ratios  
  - Significantly higher cumulative returns  

- This indicates that:
  > The relationship between assets is **time-varying**, and static hedge ratios are insufficient.

- The Kalman Filter successfully adapts to changing market dynamics, capturing profitable opportunities missed by OLS.

### Important Insight:
After applying log transformation and stricter filtering, weaker pairs were eliminated, suggesting earlier results likely contained false positives.
---

## 📊 Visualizations

1. **Z-score with trading signals**
2. **Dynamic β(t) vs static OLS β**
3. **Equity curves (OLS vs Kalman)**

---

## ⚠️ Limitations

* Sharpe ratio remains moderate (< 1)
* Returns not fully volatility-normalized
* Strategy sensitive to parameter tuning (Q)
* Limited to single-pair evaluation

---

## 🚀 Future Improvements

* Volatility scaling / position sizing
* Multi-pair portfolio construction
* More robust Kalman tuning (MLE / EM)
* Slippage & realistic execution modeling
* Risk management (drawdown control)

---

## 🛠️ Tech Stack

* Python
* NumPy, Pandas
* Statsmodels
* PyKalman
* Matplotlib

---

## 👤 Author

**Kishankumar Patel**
Master’s in Data Science & Analytics
Aspiring Quantitative Analyst / Researcher

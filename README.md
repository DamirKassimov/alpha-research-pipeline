# Systematic Alpha Research Pipeline
### Cross-Sectional Factor Signal Testing, Portfolio Optimisation, and Performance Attribution

---

## Project Objective

This project builds a full systematic equity research pipeline — from raw market data to backtested performance attribution — to investigate whether cross-sectional factor signals can generate risk-adjusted returns in excess of a passive benchmark. The core question is simple: do measurable characteristics of stocks, such as profitability, momentum, and volatility, reliably predict which stocks will outperform over the following month?

The pipeline tests ten signals across a 50-stock S&P 500 universe, constructs a dollar-neutral, sector-neutral long/short portfolio using constrained quadratic optimisation, and evaluates the result against the Fama-French five-factor model. Every step is transparent, reproducible, and grounded in peer-reviewed quantitative finance literature.

---

## Methodology

The project follows the standard institutional research workflow used by systematic equity funds.

### 1. Data Preparation
- **Universe:** 50 large-cap S&P 500 constituents across 11 GICS sectors
- **Price data:** Daily adjusted close prices and volume, sourced from Yahoo Finance via `yfinance`
- **Fundamentals:** Current snapshot of book-to-price, return on equity, earnings yield, and related ratios from Yahoo Finance (noted limitation — see below)
- **Benchmark:** Fama-French five-factor daily returns from the Ken French Data Library via `pandas-datareader`
- **Backtest period:** January 2020 to December 2024

### 2. Signal Construction

Ten cross-sectional alpha signals were computed from historical price and fundamental data. Each signal is defined so that a higher value corresponds to a more attractive stock, and each is z-scored cross-sectionally before use.

| Signal | Category | Source |
|--------|----------|--------|
| 12-1 Month Momentum | Momentum | Jegadeesh & Titman (1993) |
| 3-Month Momentum | Momentum | — |
| Short-Term Reversal | Reversal | Jegadeesh (1990) |
| Idiosyncratic Volatility | Low-Vol | Ang, Hodrick, Xing & Zhang (2006) |
| Market Beta (Low Beta) | Low-Vol | Frazzini & Pedersen (2014) |
| 52-Week High Ratio | Momentum | George & Hwang (2004) |
| Volume Trend | Technical | Lee & Swaminathan (2000) |
| Book-to-Price | Value | Fama & French (1992) |
| Return on Equity | Quality | Novy-Marx (2013) |
| Earnings Yield | Value | — |

A weighted composite score is built from signals with positive information coefficients only. Signals with negative IC are excluded from the portfolio. The IC-weighted composite achieved an ICIR of 5.070, compared to the best individual signal (ROE) at 3.048 — demonstrating that signal diversification meaningfully improves predictive consistency.

### 3. Portfolio Formation
- **Optimiser:** Constrained quadratic programme solved via CVXPY (CLARABEL solver)
- **Objective:** Maximise `α'w − λ·w'Σw` where `α` is the composite signal vector and `Σ` is the Ledoit-Wolf shrunk covariance matrix
- **Constraints:** Dollar-neutral (`Σwᵢ = 0`), sector-neutral (within each sector), gross leverage ≤ 2×, maximum 8% per position
- **Rebalancing:** Monthly (month-end), walk-forward with no look-ahead
- **Transaction costs:** 10 basis points one-way, applied to net turnover

### 4. Backtesting
- Walk-forward design: signals and covariance are estimated using only data available at the time of each rebalancing date
- Portfolio daily P&L is computed over each holding period
- Transaction costs are deducted on the first day of each new period

### 5. Performance Evaluation
- Sharpe ratio, CAGR, Sortino ratio, Calmar ratio, max drawdown, and monthly return heatmap
- Comparison against SPY (passive benchmark) and equal-weight universe portfolio

### 6. Fama-French Factor Attribution
- OLS regression of daily excess strategy returns on five factors: Market (Mkt-RF), Size (SMB), Value (HML), Profitability (RMW), and Investment (CMA)
- Newey-West HAC standard errors applied to account for return autocorrelation
- Annualised alpha and t-statistic extracted to assess whether residual returns are statistically meaningful

### 7. Signal Diagnostics
- Information Coefficient (IC): Spearman rank correlation between each signal and 21-day forward returns
- Information Coefficient Information Ratio (ICIR): IC / std(IC) × √252
- Hit rate: percentage of months where IC was positive
- IC decay curves computed at horizons of 1, 5, 10, 21, 42, and 63 trading days

### 8. Bias Checks
- Return autocorrelation tested at lags 1 through 21 against a 95% confidence interval (±0.056)
- Positive autocorrelation at lag 1 would indicate look-ahead bias — none was found
- Return distribution tested for normality via Q-Q plot (R² = 0.986) and kurtosis measurement

---

## Key Findings

### Performance Summary (January 2020 – December 2024)

| Metric | Alpha Strategy (L/S, net TC) | SPY Buy-and-Hold |
|--------|------------------------------|------------------|
| Sharpe Ratio | 0.721 | 0.703 |
| Sortino Ratio | **1.078** | 0.857 |
| Calmar Ratio | **0.608** | 0.368 |
| CAGR | 8.11% | 13.43% |
| Annualised Volatility | **11.76%** | 21.11% |
| Max Drawdown | **-13.34%** | -36.47% |
| Mean Rolling 6M Sharpe | 0.64 | — |
| Avg Monthly TC (one-way) | 10.6 bps | — |

The strategy produced a marginally higher Sharpe ratio than SPY while running at roughly half the volatility (11.76% vs 21.11%) and less than half the maximum drawdown (-13.34% vs -36.47%). The lower CAGR is expected — a dollar-neutral strategy carries no net market exposure, so it does not ride the broad equity rally of 2020–2024. The Sortino ratio of 1.078 versus SPY's 0.857 and Calmar ratio of 0.608 versus SPY's 0.368 both confirm that on a downside-adjusted basis the strategy outperforms the passive benchmark.

![Performance Dashboard](images/performance_dashboard.png)

![Strategy Performance Factsheet](images/factsheet.png)

### Fama-French Five-Factor Attribution

| Metric | Value |
|--------|-------|
| Annualised Alpha | +4.43%/yr |
| Alpha t-statistic | +0.953 |
| FF5 R² | 0.153 |

| Factor | Beta |
|--------|------|
| Mkt-RF (Market) | +0.094 |
| SMB (Size) | -0.084 |
| HML (Value) | -0.241 |
| RMW (Profitability) | +0.070 |
| CMA (Investment) | +0.118 |

The strategy's five-factor R² of 0.153 means only 15.3% of its return variance is explained by standard market factors. The remaining 84.7% is idiosyncratic — not attributable to any of the five common factor exposures. The largest factor exposure is HML at -0.241, reflecting a structural tilt toward growth over value — consistent with the 2020–2024 market environment. Market beta (Mkt-RF = +0.094) is near zero, confirming that dollar-neutrality was largely maintained throughout the backtest. The positive RMW exposure (+0.070) aligns with ROE being the strongest individual signal.

The alpha of 4.43% per year remains statistically inconclusive at a t-statistic of 0.953, primarily due to the limited five-year sample window. Reaching a t-statistic of 1.96 for an alpha of this magnitude would require approximately 8–12 years of return history — a well-documented problem in quantitative finance that affects live fund track records equally.

### Signal IC and ICIR Results (21-day horizon)

**Signals that contributed positively:**

| Signal | IC | ICIR | Hit Rate |
|--------|----|------|----------|
| Return on Equity (`roe`) | +0.0396 | +3.048 | 57.0% |
| Short-Term Reversal (`reversal`) | +0.0265 | +1.840 | 51.9% |
| Volume Trend (`vol_trend`) | +0.0178 | +1.785 | 54.4% |
| 12-1 Momentum (`mom_12_1`) | +0.0177 | +1.027 | 56.8% |
| 3-Month Momentum (`mom_3m`) | +0.0058 | +0.339 | 52.3% |

**Signals that hurt performance:**

| Signal | IC | ICIR | Hit Rate |
|--------|----|------|----------|
| Idiosyncratic Volatility (`idio_vol`) | -0.0703 | -5.320 | 36.4% |
| Book-to-Price (`book_to_p`) | -0.0548 | -4.721 | 41.1% |
| Realised Volatility (`low_vol`) | -0.0701 | -4.053 | 38.9% |
| Market Beta (`low_beta`) | -0.0750 | -3.905 | 38.2% |
| 52-Week High Ratio (`high_52w`) | -0.0469 | -2.631 | 46.8% |

Average IC across all ten signals: **-0.0210**

![Signal Evaluation Dashboard](images/signal_evaluation.png)

### Autocorrelation and Bias Check

- Lag 1 autocorrelation: within 95% confidence band — **no look-ahead bias detected**
- Minor negative autocorrelations at lags 3 and 13, consistent with short-term mean reversion and the monthly rebalancing cycle respectively
- Return distribution: skew = -0.19 (near-symmetric), excess kurtosis = 1.52 (mild fat tails), Q-Q R² = 0.986 (approximately normal)
- 1% daily Value-at-Risk: -1.89%

![Return Distribution Analysis](images/return_distribution.png)

### Portfolio Construction Diagnostics

- Mean active positions per rebalance: **27 out of 50 stocks**
- Gross exposure: flat at **2.0× throughout** — optimiser consistently hits the leverage constraint
- Net exposure: flat at **0.0×** — dollar-neutrality enforced at every rebalance

![Portfolio Characteristics](images/portfolio_characteristics.png)

---

## Interpretation

The strategy largely behaved as expected. On a pure risk-adjusted basis it matched SPY. More importantly, it achieved this with a maximum drawdown of -13.34% versus -36.47% for SPY — a meaningful advantage for any investor concerned with capital preservation. The Sortino and Calmar ratios both confirm that the strategy's edge is concentrated in its downside characteristics, not its raw return.

The most striking finding is which signals worked and which did not. Return on equity was the standout predictor, with an ICIR of +3.048 and a hit rate of 57%. It was consistent, month after month. Short-term reversal also contributed reliably. These are not surprising results in isolation — profitability and mean reversion are two of the most robust effects documented in academic literature.

What is more interesting is the failure of the low-volatility and value signals. Low beta, idiosyncratic volatility, and book-to-price all had deeply negative ICIR values, meaning they pointed in the wrong direction consistently throughout the backtest period. This is a direct consequence of the 2020–2024 market environment, dominated by high-growth, high-volatility technology companies. In that regime, buying low-volatility and cheap stocks was exactly the wrong positioning. These signals are not broken — they do not work in every regime, and this backtest period happened to be one of their worst on record.

The IC-weighting step handled this correctly. By excluding all five negative-IC signals from the composite, the pipeline automatically concentrated the portfolio in the signals that were actually working. The composite ICIR of 5.070 — significantly above any individual signal — demonstrates that combining imperfectly correlated signals reduces noise while preserving predictive power.

The alpha t-statistic of 0.953 is honest. Five years of data is not enough to statistically confirm 4.43% annualised alpha, and claiming otherwise would be misleading. What the results do support is a strategy that controls drawdowns well, filters harmful signals through IC diagnostics, and concentrates bets on quality and mean-reversion effects with strong theoretical foundations.

---

## Limitations

**Survivorship bias.** The universe is constructed from current S&P 500 membership. Companies removed from the index during the backtest period — often following poor performance or financial distress — are excluded. This inflates returns relative to what a live strategy would have achieved.

**Fundamental data is not point-in-time.** Balance sheet and profitability data sourced from `yfinance` reflects current values, not the values available at each historical rebalancing date. In a production system, this would be replaced with a Compustat or Bloomberg point-in-time dataset.

**Short sample period.** Five years is genuinely short for this type of research. The 2020–2024 window encompasses a pandemic crash, a historic bull market, an aggressive rate-hiking cycle, and an AI-driven equity rally. It is one distinct regime, not a representative sample of market conditions.

**Implicit growth tilt.** The strategy carries a negative HML exposure of -0.241, meaning it is structurally short value and long growth. Part of its performance during 2020–2024 may reflect the growth factor premium specific to that period rather than pure signal alpha.

**Transaction cost model is simplified.** A flat 10 basis points one-way does not capture the true market impact of trading at institutional scale, intraday bid-ask dynamics, or the cost of rebalancing into illiquid positions.

**Universe is small.** Fifty stocks is narrow. A production strategy would run across thousands of names in global equity markets. With fifty stocks, idiosyncratic risk at the single-name level is higher and diversification benefits are limited.

**Factor crowding.** Quality and momentum signals are widely used by systematic funds. If many participants run similar strategies, alpha from these signals compresses over time. This is not observable in a backtest.

---

## How to Run the Notebook

The project runs entirely in Google Colab. No local setup is required.

1. Open [Google Colab](https://colab.research.google.com)
2. Go to `File → Upload notebook`
3. Select `alpha_pipeline.ipynb` from this repository
4. Go to `Runtime → Run all` — or run cells one by one from top to bottom
5. Total runtime is approximately **10 minutes** on a free Colab instance

All required libraries are installed automatically in Cell 2. An active internet connection is required throughout, as the notebook fetches live data from Yahoo Finance and the Ken French Data Library.

> **Note:** Because the notebook uses live data, results will differ slightly each time it is run as market data updates. The findings documented in this README reflect a run completed on data through December 2024.

### Required Libraries

```
yfinance
pandas-datareader
cvxpy
scipy
scikit-learn
statsmodels
matplotlib
seaborn
numpy
pandas
```

All are installed via `%pip install` in the first code cell of the notebook.

---

## Technologies Used

| Tool | Purpose |
|------|---------|
| Python 3.10 | Core language |
| Google Colab | Notebook environment |
| `yfinance` | Price, volume, and fundamental data |
| `pandas-datareader` | Fama-French factor data |
| `cvxpy` | Quadratic portfolio optimisation |
| `scikit-learn` | Ledoit-Wolf covariance shrinkage |
| `statsmodels` | OLS regression and Newey-West standard errors |
| `scipy` | Spearman rank correlation, statistical tests |
| `numpy` / `pandas` | Numerical computing and data manipulation |
| `matplotlib` / `seaborn` | Visualisation |

---

## Future Improvements

Several extensions would make this research more robust and closer to production quality.

- **Extend the backtest period.** Running the pipeline on 15–20 years of data would allow statistical significance of the alpha to be properly assessed and would capture multiple market regimes including the 2008 financial crisis, the 2010–2018 low-volatility bull market, and the 2022 inflation shock.

- **Point-in-time fundamental data.** Replacing the current `yfinance` snapshot with a proper PIT database (Compustat via WRDS, or a commercial provider) would eliminate the look-ahead bias in fundamental signals and produce a cleaner backtest.

- **Expand the signal universe.** Ten signals is a starting point. Adding earnings revision signals, short interest, option-implied volatility, and text-based signals from SEC filings would diversify the alpha sources and reduce dependence on any single signal.

- **Survivorship-bias-free universe.** Using a dynamic constituent list that reflects actual index membership at each historical date would remove performance inflation and produce more realistic return estimates.

- **Alternative portfolio construction methods.** Comparing the CVXPY QP approach against Hierarchical Risk Parity (HRP) and simple quintile long/short portfolios would clarify how much performance comes from signal quality versus optimisation methodology.

- **Regime-conditional signal weighting.** The IC analysis showed that value and low-volatility signals perform very differently across regimes. A regime-detection layer — for example using a Hidden Markov Model on macro factors — could dynamically adjust signal weights based on the current environment, reducing the damage from regime shifts like 2022.

- **Out-of-sample validation.** The current backtest uses the same data to both select signals via IC weighting and evaluate performance. A proper out-of-sample test would fix signal weights using pre-2020 data and evaluate strictly on 2020–2024 data, avoiding in-sample selection bias.

---

## Academic References

- Jegadeesh, N., & Titman, S. (1993). Returns to Buying Winners and Selling Losers. *Journal of Finance*, 48(1), 65–91.
- Fama, E. F., & French, K. R. (1993). Common Risk Factors in the Returns on Stocks and Bonds. *Journal of Financial Economics*, 33(1), 3–56.
- Ang, A., Hodrick, R. J., Xing, Y., & Zhang, X. (2006). The Cross-Section of Volatility and Expected Returns. *Journal of Finance*, 61(1), 259–299.
- Frazzini, A., & Pedersen, L. H. (2014). Betting Against Beta. *Journal of Financial Economics*, 111(1), 1–23.
- Novy-Marx, R. (2013). The Other Side of Value: The Gross Profitability Premium. *Journal of Financial Economics*, 108(1), 1–28.
- Lopez de Prado, M. (2018). *Advances in Financial Machine Learning*. Wiley.
- Ledoit, O., & Wolf, M. (2004). A Well-Conditioned Estimator for Large-Dimensional Covariance Matrices. *Journal of Multivariate Analysis*, 88(2), 365–411.

---

*Built using Python and Google Colab. All data sourced from publicly available APIs.*

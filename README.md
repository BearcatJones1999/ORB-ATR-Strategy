# Opening Range Breakout (ORB) Strategy

An intraday trading strategy that trades breakouts of the first 5-minute bar on U.S. equities and leveraged ETFs, with fixed-R and ATR-based stops, end-of-day exits, and multi-symbol basket construction. Benchmarked against buy-and-hold and evaluated with factor regressions, multiple-testing corrections, slippage sensitivity, and out-of-sample tests.

Developed as a personal research project to build a realistic intraday backtesting infrastructure and stress-test a simple breakout rule against common pitfalls: execution costs, parameter tuning, and the false precision of leveraged-ETF Sharpe ratios.

## Strategy Construction

### 1. Signal
Each trading day, the first 5-minute candle (9:30–9:35 ET) determines the day's directional bias:
- Green candle (close > open) → long bias
- Red candle (close < open) → short bias
- Doji → no trade

Entry is the open of the second 5-minute bar (9:35 ET).

### 2. Stops and Exits
Two variants are implemented:

**Fixed-R ORB** — stop placed at the first bar's low (longs) or high (shorts). Profit target set at 10R (rarely hit before EOD).

**ATR-ORB** — stop placed at entry ± (ATR_fraction × 14-day ATR). ATR fraction tuned on the training sample with values tested from 3% to 10%.

Both variants exit at 16:00 ET if the stop has not been triggered.

### 3. Position Sizing
- 1% of equity risked per trade (account_risk / R = share count)
- Capped at 4x leverage (max 4x equity notional per position)
- Commission: $0.0005 per share per side
- Slippage: sensitivity-tested from 0 to 5 bps per side

### 4. Universe
- **Single-symbol backtest**: TQQQ, 2016–2025
- **Basket backtest**: 9 leveraged ETFs — TQQQ, TECL, BULZ, SSO, URTY, GGLL, TSLL, NVDL, MSTX — equal-weighted, 2020–2025

## Evaluation Framework

- **CAPM** and **Fama–French 5-factor** regressions with HAC (Newey–West, 5–6 lag) standard errors
- **Train/test splits**: parameters tuned on 2020–2022, evaluated on 2023–2025
- **Multiple-testing corrections**: Bonferroni, Benjamini–Hochberg (FDR), Benjamini–Yekutieli (FDR under dependence)
- **Slippage sensitivity**: 0, 1, 2, 3, and 5 bps per side
- **Drawdown analysis**: top-5 drawdown episodes with peak-to-trough timing

## Results

### Single-Symbol: TQQQ ORB vs Buy & Hold (2016–2025, 2,490 trading days)

| Metric | ORB | Buy & Hold |
|---|---|---|
| CAGR | **36.50%** | −6.55% |
| Annual Volatility | 37.99% | 76.65% |
| Sharpe | **0.96** | −0.09 |
| Max Drawdown | **−37.55%** | −91.88% |
| Total Return | 2,064% | −48.79% |

Buy-and-hold TQQQ looks poor because the sample straddles multiple deep drawdowns (volatility decay + leverage punishes passive holders). The ORB variant avoids this by being flat overnight.

**CAPM regression (HAC)**: α = 0.15%/day (t = 3.26), β ≈ 0 on buy-and-hold TQQQ. Annualized alpha ≈ 38%. R² ≈ 0, indicating ORB returns are effectively uncorrelated with the passive benchmark.

### Multi-Symbol Basket: Simple ORB (2020–2025, 1,486 trading days)

| Symbol | CAGR | Vol | Sharpe | MaxDD |
|---|---|---|---|---|
| TQQQ | 25.48% | 36.92% | 0.69 | −37.54% |
| TECL | 21.20% | 35.41% | 0.60 | −43.70% |
| BULZ | 24.09% | 30.68% | 0.79 | −42.42% |
| SSO | 9.19% | 34.42% | 0.27 | −43.27% |
| URTY | 29.69% | 37.52% | 0.79 | −48.51% |
| GGLL | 27.57% | 24.08% | 1.14 | −33.45% |
| TSLL | 18.98% | 23.45% | 0.81 | −28.36% |
| NVDL | 15.23% | 22.47% | 0.68 | −22.63% |
| MSTX | 4.70% | 16.99% | 0.28 | −32.97% |
| **Portfolio (EW)** | **23.19%** | **14.95%** | **1.55** | **−14.75%** |

Equal-weighting across the basket roughly halves volatility (15% vs 25–37% on individual names) because ORB returns across symbols are only moderately correlated (average pairwise correlation 0.19, max 0.72).

**Train/Test (simple ORB)**: Train Sharpe 1.54 (2020–2022), Test Sharpe 1.58 (2023–2025). Stable across the split.

### Multi-Symbol Basket: ATR-ORB with Tuned Stops (2023–2025 test period)

| Symbol | CAGR | Sharpe | MaxDD |
|---|---|---|---|
| TQQQ | 77.75% | 2.04 | −21.26% |
| TECL | 123.03% | 2.40 | −25.37% |
| BULZ | 253.78% | 4.46 | −22.04% |
| SSO | −7.25% | −0.47 | −30.70% |
| URTY | 60.02% | 1.51 | −29.06% |
| GGLL | 215.47% | 3.75 | −13.35% |
| TSLL | 367.15% | 5.20 | −19.35% |
| NVDL | 271.59% | 4.18 | −23.14% |
| MSTX | 46.69% | 0.94 | −42.51% |
| **Portfolio (EW)** | **149.08%** | **5.78** | **−8.36%** |

**Portfolio win rate: 43.72%** on trade days (win rate at the daily portfolio level, not per-trade). Individual symbol win rates range from 10.6% (SSO) to 17.1% (GGLL), with the basket benefiting from asymmetric payoffs (few large winners, many small losers capped at −1R).

### Fama–French 5-Factor Regression (ATR-ORB Basket, Test Period)

| Factor | Loading | t-stat |
|---|---|---|
| Mkt-RF | −0.05 | −0.29 |
| SMB | −0.07 | −0.49 |
| HML | 0.06 | 0.47 |
| RMW | 0.00 | 0.01 |
| **α (daily)** | **0.34%** | **5.61** |

Annualized alpha ≈ 85%. All factor loadings are statistically indistinguishable from zero. The strategy does not behave like an equity factor bet. It's closer to an independent return stream. R² ≈ 0.007.

### Multiple-Testing Corrections

Testing whether each symbol's mean excess return is significantly positive:

| Significance Test | Significant Symbols (of 9) |
|---|---|
| Raw (p < 0.05) | 7 |
| Bonferroni | 5 |
| Benjamini–Hochberg (FDR) | 7 |
| Benjamini–Yekutieli (FDR, dep.) | 6 |

Even under Bonferroni's conservative correction, 5 of 9 names retain significance. BULZ, TSLL, NVDL, GGLL, and TECL survive all corrections.

### Slippage Sensitivity (Single-Symbol ORB)

| Slippage (per side) | CAGR | Sharpe | Total Return |
|---|---|---|---|
| 0 bps | 65.4% | 1.65 | 14,364% |
| 1 bp | 50.5% | 1.28 | 5,589% |
| 2 bps | 36.9% | 0.93 | 2,133% |
| 3 bps | 24.5% | 0.62 | 775% |
| 5 bps | **−51.6%** | **−2.72** | **−99.9%** |

**This is the most important table in the analysis.** A 2 bps-per-side assumption, which is realistic for liquid ETFs at market open keeps the strategy profitable. At 5 bps, the strategy is destroyed. Real-world execution risk is the dominant threat, not signal decay.

## Honest Caveats

The ATR-ORB basket produces a **5.78 Sharpe** on the test sample. This number deserves aggressive scrutiny, and the reader should not take it at face value. Specific concerns:

1. **Leveraged ETFs inflate raw Sharpe**. A 5.78 Sharpe on 3x-leveraged instruments is not comparable to a 5.78 on SPY. The underlying volatility base is 3x larger, so the same alpha translates to a 3x larger Sharpe numerator relative to the Sharpe of the underlying signal on unlevered equities.

2. **Parameter selection bias**. ATR_FRAC was chosen on the training sample to maximize Sharpe (3% selected with train Sharpe of 7.83). The reported test Sharpe reflects this selection. A nested cross-validation would yield a more defensible estimate.

3. **Slippage is the binding constraint**. At 5 bps per side, returns are fully wiped out. Leveraged ETFs have wider spreads and thinner books than SPY, especially in the first minutes of the session. The 2 bps assumption is plausible but not guaranteed.

4. **Leveraged ETF path dependence**. Daily-rebalanced leverage introduces volatility decay that a 5-minute backtest on intraday bars may underrepresent, particularly on gapping days.

5. **Survivorship bias in universe selection**. The 9-symbol basket was chosen after the fact. Several of these ETFs (GGLL, MSTX, NVDL, BULZ) were launched recently; their early-sample absence isn't survivorship per se, but their *inclusion* as a basket was informed by knowing they exist.

6. **No live trading verification**. All results are backtested. Queue position, partial fills, opening auction dynamics, and halts are not modeled.

The point of this project is not to claim a production-ready strategy. It's to:
- Build a realistic intraday backtesting infrastructure
- Apply correct inference (HAC, FDR corrections, factor-adjusted alpha)
- Honestly characterize where the edge breaks down (slippage, ETF-specific risk)
- Practice the discipline of treating impressive-looking results skeptically

## Limitations & Extensions

- **No live execution testing** — paper-trading validation on Alpaca is a natural next step
- **Single-regime test period** — 2023–2025 was a strong-trend market; bear/chop regimes unknown
- **Position sizing is static** — Kelly fractional or volatility-targeted sizing could improve risk-adjusted returns
- **No regime filtering** — the signal fires on every non-doji day; filtering by VIX regime, overnight gap size, or opening volume may help
- **Basket construction is naive** — equal weights; risk-parity or inverse-variance weighting would likely improve Sharpe further but also requires walk-forward validation

## How to read this repo

- **Executive Summary** — key findings, equity curves, and the main caveats in one page
- **In-Depth Analysis** — full methodology, per-symbol tables, regression output, slippage sensitivity, and drawdown episodes

## Stack

Python, pandas, NumPy, statsmodels, matplotlib. Data from the Alpaca market data API (5-minute and daily bars). API credentials should be loaded from environment variables  never committed to source control.

---

*Educational project. Not investment advice. Leveraged ETFs carry substantial decay risk and are not suitable for buy-and-hold or most retail strategies.*

Opening Range Breakout (ORB) – Multi-Asset Intraday Strategy Research

This repository contains a complete research pipeline for developing, testing, and evaluating intraday Opening Range Breakout (ORB) strategies across a diversified ETF basket. The project explores three iterations of the ORB strategy, integrating realistic execution modeling, ATR-based volatility controls, walk-forward optimization, and detailed performance analytics.

Project Overview

The Opening Range Breakout strategy attempts to exploit the concentrated burst of price discovery occurring during the first few minutes of the U.S. trading session.

This project evaluates and evolves the ORB framework through:

1. A classic breakout with a fixed 10R profit target

2. A volatility-adjusted ORB using ATR-based stop-loss sizing

3. An ATR-based trailing-stop ORB with dynamic intraday risk management

The strategies are tested across a basket of leveraged ETFs and ETNs using Alpaca market data.

Strategy Iterations

1. ORB Version 1 — Fixed Stop and 10R Profit Target

This is the baseline ORB model:

Direction determined by the first 5-minute candle

Entry at the open of the second candle

Stop-loss at the high/low of the first candle

Profit target = 10 × initial risk (10R)

Exit at stop, profit target, or end-of-day

No ATR or trailing behavior

This version is used to benchmark the classic ORB payoff structure.


2. ORB Version 2 — ATR-Based Stop, No Profit Cap

This version introduces volatility-adjusted risk sizing:

ATR(14) derived from daily bars

Stop distance = ATR_FRAC × ATR(14)

1% risk per trade, capped by 2× leverage

Slippage (bps) and commissions included

Trades exit only by ATR stop or end-of-day

No profit target

This standardizes risk across assets with different volatility characteristics.


3. ORB Version 3 — ATR-Based Trailing Stop

This enhanced model introduces dynamic stop tightening:

Same ATR sizing as Version 2

Stop automatically “ratchets” closer as price moves favorably

Long trades: stop follows rising lows

Short trades: stop follows falling highs

Exit at trailing stop or end-of-day

This structure captures larger intraday moves while limiting intraday drawdowns.

Data Pipeline

The notebook uses Alpaca’s StockHistoricalDataClient to fetch:

Daily OHLC Bars

Used to compute:

True Range

ATR(14)

ATR-based risk sizing

5-Minute Intraday Bars

Used to define:

ORB direction

Entry and stop levels

Intraday stop/target detection

End-of-day exits

All intraday data is timezone-adjusted and restricted to regular trading hours (09:30–16:00 ET).

Performance Metrics

For each strategy and portfolio configuration, the following metrics are computed:

CAGR

Annualized volatility

Sharpe ratio

Maximum drawdown

Total return

Daily equity curve

A multi-asset portfolio is constructed by equally weighting returns across all symbols.

Walk-Forward ATR Optimization

To mitigate the risk of overfitting, the notebook includes a walk-forward optimization process:

Split the dataset into training and testing windows

On each training window, evaluate a set of ATR_FRAC candidates

Select the ATR_FRAC with highest Sharpe

Apply that ATR_FRAC to the next test window

Concatenate all out-of-sample test windows into a unified equity curve

Outputs include the best ATR_FRAC per period and full OOS performance statistics.

Key Findings

These points can be adjusted based on your actual results:

Opening Range Breakout behavior is strongest in volatile, leveraged ETFs

ATR-based stop sizing significantly improves consistency across assets

Trailing-stop variants further reduce drawdowns while preserving upside capture

Slippage and commissions materially reduce returns; realistic modeling is essential

Walk-forward results suggest robustness across market regimes

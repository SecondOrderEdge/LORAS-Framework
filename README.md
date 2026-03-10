# LORAS "Home Run" Strategy — Crypto Momentum Fact Sheet

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Python 3.9+](https://img.shields.io/badge/python-3.9%2B-blue.svg)](https://www.python.org/)
[![Platform](https://img.shields.io/badge/platform-Colab%20%7C%20Local-lightgrey.svg)](https://colab.research.google.com/)
[![Research Only](https://img.shields.io/badge/use-research%20%2F%20educational%20only-red.svg)](#compliance--legal-disclaimer)

> **For educational and research use only. Not investment advice. Not a trading system. See [disclaimer](#compliance--legal-disclaimer) below.**

A Python notebook that downloads historical crypto price data and generates a clean, white-background **"Fact Sheet"** style research report for a simple momentum/rotation strategy across BTC, ETH, and SOL. Includes a walk-forward backtest, drawdown profile, allocation history, and a live-signal desk table.

---

## What is LORAS?

**LORAS** is a decision-making framework that mirrors the natural flow of an investment committee (IC) or CIO conversation:

| Letter | Pillar | Question it answers |
|--------|--------|---------------------|
| **L** | **Liquidity** | Do we have the liquidity to do this? (cash buffer, redemption gates, drawdowns, funding needs) |
| **O** | **Opportunity** | Is the opportunity attractive right now? (spread levels, factor exposure, macro regime) |
| **R** | **Risk** | What are the risks? (volatility, drawdown, correlation, VaR, manager risk) |
| **A** | **Allocation** | How big should the allocation be? (position sizing, diversification benefit, marginal contribution) |
| **S** | **Strategy** | How do we implement? (deployment timing, scaling plan, triggers, monitoring) |

The framework is designed to structure research output the way a portfolio manager or allocator already thinks — making it easier to move from analysis to an actionable IC recommendation.

![Strategy Fact Sheet Output](Output.png)

---

## Table of Contents

- [What is LORAS?](#what-is-loras)
- [What This Does](#what-this-does)
- [Strategy Logic](#strategy-logic)
- [Sample Output](#sample-output)
- [Quickstart](#quickstart)
- [Configuration](#configuration)
- [Known Limitations](#known-limitations)
- [Compliance & Legal Disclaimer](#compliance--legal-disclaimer)
- [Ethics & Attribution](#ethics--attribution)
- [License](#license)

---

## What This Does

1. **Fetches daily OHLCV data** for BTC-USD, ETH-USD, and SOL-USD via `yfinance`
2. **Fetches a risk-free proxy** (3-month Treasury rate) from FRED via `pandas_datareader`
3. **Computes indicators** used by the strategy:
   - Trend filter: EMA(10) vs EMA(40)
   - Breakout filter: price vs prior 20-day high (Donchian-style)
   - Momentum score: `(Price / SMA50) × (RSI / 50)`
   - Stop level: ATR-based trailing stop `ATR(14) × multiplier`
4. **Runs a walk-forward backtest** with monthly rebalancing, a BTC regime guard, and dynamic leverage
5. **Generates a single-page Fact Sheet** with performance, risk metrics, allocation history, and a live signals table

---

## Strategy Logic

### Universe
BTC-USD, ETH-USD, SOL-USD

### Rebalance Frequency
Monthly (~21 trading days)

### Momentum-Based Allocation
On each rebalance date:
```
MomentumScore = (Price / SMA50) × (RSI / 50)
```
- Scores ≤ 0 are floored to zero
- Weights set proportionally: `w_i = score_i / sum(scores)`

### BTC Guard (Regime Filter)
If BTC is below its SMA50 on a rebalance date, ETH and SOL weights are set to 0 and the blocked allocation sits in cash.

### Trend & Leverage Rules (Daily)
| Condition | Leverage |
|-----------|----------|
| Not trending (EMA10 < EMA40) | 0x (cash) |
| Trending, no breakout | 1x |
| Trending + breakout | Up to 2x (configurable) |

Signals use prior-day values to reduce lookahead bias.

### Stop Display
ATR-based stop levels are computed and displayed in the Live Trade Desk table. The backtest does **not** simulate intraday stop execution (see [Known Limitations](#known-limitations)).

---

## Sample Output

The generated Fact Sheet contains five panels:

| Panel | Description |
|-------|-------------|
| Header | Plain-English methodology summary |
| Performance | Cumulative returns (log scale) vs BTC benchmark |
| Drawdown | Underwater equity curve |
| Allocation | Stacked historical weight chart |
| Live Trade Desk | Current price, allocation %, leverage state, stop price, status |

**Backtest metrics shown:** CAGR, Annualized Volatility, Sharpe Ratio, Max Drawdown

---

## Quickstart

### Option A — Google Colab (Recommended)

1. Open the `.ipynb` file in [Google Colab](https://colab.research.google.com/)
2. Run all cells — the notebook auto-installs its dependencies
3. The Fact Sheet renders inline

### Option B — Local Python (3.9+)

```bash
# Install dependencies
pip install -r requirements.txt

# Launch the notebook
jupyter notebook bitcoin_eth_sol_momentum_strategy_backtest.ipynb
```

---

## Configuration

All parameters live in the `Config` class at the top of the notebook:

| Parameter | Default | Description |
|-----------|---------|-------------|
| `TICKERS` | `['BTC-USD', 'ETH-USD', 'SOL-USD']` | Assets to include |
| `START_DATE` | `'2020-01-01'` | Backtest start date |
| `MAX_LEVERAGE` | `2.0` | Maximum leverage multiplier |
| `ATR_PERIOD` | `14` | Lookback for ATR calculation |
| `ATR_MULTIPLIER` | `2.0` | Multiplier for stop distance |
| `TRAIN_WINDOW_YEARS` | `1` | Warm-up window before simulation starts |
| `TEST_WINDOW_MONTHS` | `1` | Rebalance frequency |

---

## Known Limitations

This project is intentionally simplified for research illustration. Before drawing any conclusions:

1. **Data quality** — Yahoo Finance crypto data may contain missing values, timing quirks, and exchange-specific differences. `Adj Close` behavior for crypto differs from equities.
2. **Lookahead bias** — The script attempts to use prior-day signals for daily decisions, but does not model realistic execution.
3. **No slippage or market impact** — Bid/ask spreads, market impact, and funding rates are not modeled.
4. **Transaction costs** — Uses a simplified daily drag proxy (`abs(pos) × 0.0001`), not a true cost model.
5. **Stops not simulated** — ATR stops are calculated and displayed but are not applied to the backtest P&L.
6. **Risk-free proxy** — 3-month Treasury rate from FRED, resampled to a daily approximation.
7. **Sharpe simplification** — Annualized mean / annualized volatility; does not fully integrate per-step excess returns.
8. **Leverage mechanics** — The leverage multiplier is arithmetic only; no margin requirements, liquidation risk, borrow costs, or exchange mechanics are modeled.
9. **Universe selection bias** — BTC, ETH, and SOL are survivors. Their historical availability and liquidity varied significantly over the backtest window.
10. **No out-of-sample validation** — Results should not be interpreted as predictive of future performance.
11. **Rebalance timing** — Monthly cadence is a simplification; actual optimal rebalance frequency is unknown.

---

## Compliance & Legal Disclaimer

### Not Investment Advice
This code and its outputs are provided **for informational and educational purposes only**. Nothing in this repository constitutes investment, legal, tax, or accounting advice, and nothing is a recommendation to buy, sell, or hold any security, digital asset, or investment product.

### No Client Relationship / No Fiduciary Duty
Use of this repository does not create an advisory relationship of any kind. No fiduciary duty is assumed or implied.

### No Solicitation / No Offer
This repository does not constitute an offer, solicitation, or marketing of advisory services, securities, or investment strategies of any kind.

### Risk Warning
Digital assets are highly volatile and can experience rapid, material losses. **Past backtest results do not predict future performance.** Backtests are subject to look-ahead bias, survivorship bias, and overfitting — even when steps are taken to reduce them.

### Limitation of Liability
To the maximum extent permitted by applicable law, the author(s) and any affiliated firm disclaim all liability for any direct, indirect, incidental, or consequential losses arising from the use of this code, its outputs, or reliance on any information contained herein. **Use entirely at your own risk.**

### Your Responsibility
You are solely responsible for validating assumptions, data quality, regulatory compliance, suitability, and implementation details before using anything derived from this repository.

---

## Ethics & Attribution

If you publish results derived from this code, please:

- Disclose all assumptions and limitations
- Avoid presenting backtests as real or live performance
- Provide full methodology, date ranges, and universe construction details
- Cite data sources: [Yahoo Finance](https://finance.yahoo.com/) and [FRED (Federal Reserve Bank of St. Louis)](https://fred.stlouisfed.org/)

---

## License

This project is licensed under the **MIT License** — see [LICENSE](LICENSE) for full terms.

The MIT License permits reuse, modification, and distribution, but provides **no warranty** and limits liability. The legal disclaimers in this README are supplementary to, not a replacement for, the LICENSE terms.

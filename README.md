# Hedge Fund Quant Pipeline

A production-style quantitative research pipeline that walks through the complete workflow of a systematic hedge fund: from raw price data ingestion through feature engineering, signal generation, portfolio construction, risk management, and full backtesting. The system implements a dollar-neutral long/short equity strategy using a composite signal derived from momentum, mean-reversion, technical, and volatility features.

---

## Table of Contents

- [Overview](#overview)
- [Pipeline Architecture](#pipeline-architecture)
- [Universe and Parameters](#universe-and-parameters)
- [Requirements](#requirements)
- [Installation](#installation)
- [Usage](#usage)
- [Stage Breakdown](#stage-breakdown)
- [Signal Construction](#signal-construction)
- [Portfolio Construction](#portfolio-construction)
- [Risk Management](#risk-management)
- [Backtest Results](#backtest-results)
- [License](#license)

---

## Overview

This notebook replicates the end-to-end research process used at quantitative hedge funds. It demonstrates how raw market data is transformed into actionable portfolio decisions through a rigorous, reproducible pipeline with proper handling of transaction costs, risk metrics, and benchmark comparison.

| Parameter        | Value                                                       |
| ---------------- | ----------------------------------------------------------- |
| Strategy         | Dollar-neutral long/short equity                            |
| Universe         | AAPL, MSFT, GOOGL, AMZN, JPM, GS, XOM, CVX, SPY (benchmark) |
| Period           | 2018-01-01 to 2024-12-31                                    |
| Rebalancing      | Daily signal, smoothed weights                              |
| Transaction Cost | 10 bps per trade                                            |
| Signal           | Composite: Momentum + Z-score + RSI + Low-volatility        |

---

## Pipeline Architecture

```
Raw OHLCV Prices
       |
       v
  Returns (log + simple)
       |
       v
  Feature Engineering
       |
       v
  Composite Signal Model
       |
       v
  Portfolio Construction
  (Dollar-neutral quintile book)
       |
       v
  Risk Management
  (VaR, CVaR, Drawdown, Ratios)
       |
       v
  Execution & Backtest
  (Transaction costs applied)
       |
       v
  Performance Dashboard
```

---

## Universe and Parameters

| Ticker | Sector          | Role                 |
| ------ | --------------- | -------------------- |
| AAPL   | Technology      | Long/Short candidate |
| MSFT   | Technology      | Long/Short candidate |
| GOOGL  | Technology      | Long/Short candidate |
| AMZN   | Consumer / Tech | Long/Short candidate |
| JPM    | Financials      | Long/Short candidate |
| GS     | Financials      | Long/Short candidate |
| XOM    | Energy          | Long/Short candidate |
| CVX    | Energy          | Long/Short candidate |
| SPY    | Broad Market    | Benchmark only       |

---

## Requirements

- Python 3.9+
- NumPy
- Pandas
- Matplotlib
- yfinance

---

## Installation

1. Clone the repository:

```bash
git clone https://github.com/abrar2030/Hedge_Fund_Pipeline.git
cd Hedge_Fund_Pipeline
```

2. Install dependencies (executed automatically in the notebook):

```bash
pip install numpy pandas matplotlib yfinance
```

3. Open the Jupyter Notebook:

```bash
jupyter notebook Hedge_Fund_Pipeline.ipynb
```

---

## Usage

Run all cells in the notebook sequentially. The pipeline is fully self-contained and will:

1. Download adjusted closing prices and volume for the full universe
2. Compute log returns, simple returns, and multi-horizon cumulative returns
3. Engineer cross-sectional features: momentum, volatility, Z-score, RSI, Bollinger position
4. Rank features and combine into a weighted composite signal
5. Construct dollar-neutral long/short quintile portfolios
6. Apply position limits and turnover smoothing
7. Compute comprehensive risk metrics (VaR, CVaR, Sharpe, Sortino, Calmar)
8. Run full backtest with transaction costs
9. Generate an equity curve dashboard with benchmark comparison

---

## Stage Breakdown

### 1. Raw Price Data

Downloads adjusted OHLCV for the full universe from Yahoo Finance. Handles missing data with forward-fill and drops symbols with excessive missing values.

### 2. Returns

- Log returns for time-series models
- Simple returns for portfolio aggregation
- Multi-horizon returns: 1-month, 3-month, 12-month
- Excess returns over SPY benchmark

### 3. Feature Engineering

All features are computed cross-sectionally and ranked:

| Feature          | Construction                          | Direction                             |
| ---------------- | ------------------------------------- | ------------------------------------- |
| Momentum (12-1)  | 12-month return minus 1-month return  | Positive weight                       |
| Z-Score          | Normalised deviation from 63-day mean | Negative weight (mean reversion)      |
| RSI (14)         | Relative Strength Index               | Negative weight (overbought/oversold) |
| Volatility (21d) | 21-day realised volatility            | Negative weight (low-vol anomaly)     |

### 4. Signal Model

A weighted composite of ranked features:

| Signal Component      | Weight |
| --------------------- | ------ |
| Momentum (12-1) rank  | +0.35  |
| Z-Score rank          | -0.25  |
| RSI (14) rank         | -0.20  |
| Volatility (21d) rank | -0.20  |

The composite is cross-sectionally Z-scored to remove market drift and ensure dollar neutrality.

---

## Portfolio Construction

- **Long book**: Top quintile (20%) of signals
- **Short book**: Bottom quintile (20%) of signals
- **Max single-name exposure**: 10%
- **Weight smoothing**: 70% current signal + 30% previous weights (reduces turnover)
- **Rebalancing**: Daily with blended transitions

---

## Risk Management

The pipeline computes a full suite of risk metrics:

| Metric                | Description                                    |
| --------------------- | ---------------------------------------------- |
| Annualised Return     | Return scaled to 252 trading days              |
| Annualised Volatility | Standard deviation scaled to 252 days          |
| Sharpe Ratio          | Return per unit of total risk                  |
| Sortino Ratio         | Return per unit of downside risk               |
| Calmar Ratio          | Return per unit of maximum drawdown            |
| Maximum Drawdown      | Largest peak-to-trough decline                 |
| VaR (95%)             | Value at Risk at 95% confidence                |
| CVaR (95%)            | Conditional Value at Risk (expected shortfall) |
| Information Ratio     | Active return per unit of tracking error       |

---

## Backtest Results

The backtest applies 10 bps transaction costs per trade and produces:

- Gross vs net equity curves
- Drawdown profile over time
- Rolling Sharpe ratio (6-month window)
- Cumulative return comparison vs SPY benchmark
- Monthly return heatmap
- Trade turnover analysis

---

## License

This project is licensed under the MIT License. See [LICENSE](LICENSE) for details.

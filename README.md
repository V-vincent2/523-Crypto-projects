# Momentum Effects in Cryptocurrency (Hourly Analysis with Volatility Clustering)

A research and trading strategy project extending [Caporale & Plastun (2020)](https://doi.org/10.1016/j.frl.2019.101455) by focusing on:
- **Hourly abnormal returns** (instead of daily)
- **Volatility clustering** using ATR & GARCH
- **Momentum** vs. **Contrarian** signals in Bitcoin, Ethereum, and Solana

---

## Table of Contents
1. [Overview](#overview)
2. [Data & Preprocessing](#data--preprocessing)
3. [Methodology](#methodology)
   - [Hourly Abnormal Returns](#hourly-abnormal-returns)
   - [Volatility Clustering (ATR & GARCH)](#volatility-clustering-atr--garch)
   - [Trading Strategies](#trading-strategies)
4. [Results & Findings](#results--findings)
5. [Conclusion & Future Work](#conclusion--future-work)
6. [How to Run](#how-to-run)
7. [References](#references)

---

## Overview

Cryptocurrencies trade **24/7** across all global time zones, making a “daily” definition of momentum or abnormal return somewhat arbitrary. This project investigates **hourly** returns to:

- Identify **momentum** or **contrarian** effects after an hour of “abnormal” price movement.
- Explore whether factoring in **volatility clustering** (high-vol vs. normal-vol phases) can enhance profitability or control risk.

---

## Data & Preprocessing

- **Source**: [Coinbase Pro API](https://docs.cloud.coinbase.com/exchange/docs)
- **Symbols**: BTC-USD, ETH-USD, SOL-USD
- **Dates**: 2019-01-01 to 2024-05-20
- **Frequency**: 1-hour OHLCV data
- **Missing Data**: Interpolated to fill gaps

**Train/Test Split**:
- **Train**: 2019-01-01 to 2023-06-01
- **Test**: 2023-06-01 to 2024-05-20

```python
# Example snippet for data retrieval & interpolation
btc_df = fetch_crypto_data("BTC-USD", start_time, end_time, barSize="3600")
btc_df = btc_df.resample('H').mean()  # ensure each hour is accounted for
btc_df['close'] = btc_df['close'].interpolate(method='time')

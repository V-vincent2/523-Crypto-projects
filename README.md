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
6. [References](#references)

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

```






## Methodology

### Hourly abnormal returns
The abnormal returns are calculated using a **moving average** and **standard deviation** over a rolling window. The calculation is as follows:

- **Mean and Standard Deviation for Rolling Window**:
  
  $$\mu_t = \frac{1}{n} \sum_{i=t-n+1}^{t} R_i$$
  
  $$\sigma_t = \sqrt{\frac{1}{n-1} \sum_{i=t-n+1}^{t} (R_i - \mu_t)^2}$$
  
- **Threshold for Abnormal Returns**:
  
  $$\text{Threshold}_t = \mu_t + k \cdot \sigma_t \cdot \sqrt{\text{lookback}}$$

Where `k` is a scaling factor used to identify extreme price movements.

### Volatility Clustering
Volatility clustering refers to periods of high or low volatility, which are common in cryptocurrencies. The project uses **ATR (Average True Range)** and **GARCH(1,1)** models to measure volatility and classify high-volatility periods.

#### ATR Method
- **True Range (TR)** is defined as the maximum of:
  
  $$TR_t = \max(H_t - L_t, |H_t - C_{t-1}|, |L_t - C_{t-1}|)$$
  
  Where $H_t$, $L_t$, and $C_{t-1}$ are the high, low, and close prices, respectively.
  
- **Average True Range (ATR)** is then computed over a rolling window to capture volatility.

#### GARCH Model
A **GARCH(1,1)** model is fit on the hourly returns to estimate conditional volatility:

$$
\sigma_t^2 = \omega + \alpha \cdot \epsilon_{t-1}^2 + \beta \cdot \sigma_{t-1}^2
$$

Where:
- $\sigma_t^2$ is the conditional variance,
- $\epsilon_{t-1}$ is the lagged residual error, and
- $\sigma_{t-1}^2$ is the previous period's variance.

### Strategy
This project uses two strategies: **Momentum** and **Contrarian**, with separate rules for high and normal volatility periods.

- **Momentum Strategy**: Enter a long position when abnormal positive returns are observed, and exit after a target profit or stop-loss condition is met.
- **Contrarian Strategy**: Enter a long position after abnormal negative returns, betting on price reversals.

### Signal Process
- **Entry Signal**: 
  - A long position is taken when the cumulative returns exceed the threshold for positive or negative returns.
  
- **Exit Signal**: 
  - The position is closed after a specified number of hours or when a stop-loss or profit target is hit.

## Results & Findings

### Strategy Comparison
Below is a comparison of performance metrics for different strategies:

| Strategy            | Sharpe Ratio | Max Drawdown | Total Return (%) |
|---------------------|--------------|--------------|------------------|
| BTC (No Vol Filter) | 2.31         | 12.5%        | 57.0             |
| BTC (ATR Filter)    | 3.21         | 5.07%        | 0.17             |
| SOL (No Vol Filter) | 1.84         | 20.2%        | 206.0            |
| SOL (ATR Filter)    | -0.45        | 15.9%        | -2.3             |

#### Key Insights:
1. **Hourly vs. Daily**: 
   - Hourly abnormal returns provide more accurate momentum/contrarian signals compared to daily data, which averages out important price movements.
  
2. **Volatility Filters**:
   - **ATR-based filters** showed lower drawdowns for BTC but reduced total returns for SOL and ETH.
   - **GARCH-based models** underperformed relative to ATR due to higher volatility estimations.

3. **Momentum vs. Contrarian**:
   - Positive abnormal returns exhibited clear momentum for BTC.
   - Negative abnormal returns can be exploited using a contrarian strategy or by shorting, but results were inconsistent.

### Performance Metrics
- **Sharpe Ratio**: BTC with ATR-based filtering showed strong risk-adjusted returns.
- **Max Drawdown**: High volatility periods (measured via ATR and GARCH) tended to result in higher drawdowns.
- **Total Returns**: SOL showed the highest returns, particularly when no volatility filter was applied.

## Conclusion & Future Work

### Conclusion
- **Momentum Strategy**: Successfully captures price moves following abnormal returns, especially for BTC. The contrarian approach works well in certain market conditions but less consistently.
  
- **Volatility Clustering**: ATR-based volatility filters are useful in reducing risk, but they also reduce returns in some cases. The GARCH-based approach, while more sophisticated, didn’t significantly outperform simpler models.

### Future Work
- **Explore Alternative Volatility Models**: Investigate multifractal analysis (MF-DFA) to capture volatility clustering more effectively.
- **Machine Learning**: Implement machine learning techniques to optimize strategy parameters dynamically and potentially uncover non-linear relationships.
- **Cross-Coin Portfolio**: Combine multiple cryptocurrencies (e.g., BTC, ETH, SOL) to create a diversified portfolio and test whether the strategies can be applied across assets for better risk-adjusted returns.
- **More Granular Features**: Incorporate time-of-day, day-of-week, and seasonal features to capture temporal patterns in cryptocurrency prices.

### Future Improvements
- **Parameter Tuning**: Further fine-tune the parameters using cross-validation techniques.
- **Model Robustness**: Extend the analysis to out-of-sample data (e.g., using more recent market conditions).
- **Transaction Costs**: Incorporate more accurate models for transaction costs, particularly in high-frequency trading scenarios.

## References
1. Caporale, G., & Plastun, A. (2020). Momentum effects in the cryptocurrency market after one-day abnormal returns.
2. Mensi, W., Lee, H. Y., Al-Yahyaee, K. H., Şensoy, A., & Yoon, S. M. (2019). Intraday downward/upward multifractality and long memory in Bitcoin and Ethereum markets.
3. Li, Y., Urquhart, A., Wang, P., & Zhang, W. (2021). MAX momentum in cryptocurrency markets.

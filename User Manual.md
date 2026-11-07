Axiom Chronos Vector — User Manual
Welcome to Axiom Chronos Vector. This guide explains how to use the app, what the metrics mean, and how to interpret the results of your backtests.

1. Getting Started
The Workflow
Chronos uses an explicit "Fetch then Run" workflow to prevent accidental API rate limits and ensure you are always testing the data you see on screen.

Select Market & Symbol: Choose Crypto, Stocks, or Commodities, and type the ticker (e.g., BTCUSDT, AAPL, GC=F).

Choose Timeframe & History: Select your candle size (1m to 1d) and how many years of history you want.

Fetch Data: Tap FETCH MARKET DATA. The app will download the candles and run them through the Integrity Engine. Wait for the "LOADED VECTOR" banner to appear.

Configure Strategy: Select your strategy (e.g., EMA Cross) and adjust parameters.

Set Risk & Capital: Enter your initial capital and set your Stop Loss / Take Profit.

Run Backtest: Tap EXECUTE BACKTEST.

Free vs. Full Edition
Chronos is free to download and use. The Full Edition unlocks:

Advanced Risk Metrics (Sharpe, Sortino, Calmar, Kelly).

Monte Carlo Probability Cloud.

Volatility Strategies (Keltner, ATR Breakout).

PDF Report Export.

You can purchase the Full Edition via the Paywall screen. If you have already purchased it, tap Restore Purchase.

2. Understanding the Data (The Integrity Engine)
Chronos is built on the principle of Intellectual Honesty. If the data is bad, the app tells you.

After you fetch data, look at the Validity Rating on the Performance HUD:

HIGH: The data is clean, and you have enough trades (100+) to trust the sample.

MODERATE: The data is mostly clean, or you have between 30 and 99 trades. The results are indicative but need caution.

LOW: The sample size is too small (under 30 trades). The strategy might be overfit.

UNTRUSTWORTHY: The data provider sent missing candles, huge gaps, or duplicates. The results are mathematically invalid.

If the banner shows "PROVIDER RETURNED X OF Y REQUESTED YEARS", it means the provider (e.g., Yahoo Finance) does not have enough history for that timeframe. The backtest was run on the shorter timeframe.

3. Understanding the Metrics
Total Return: The percentage growth of your capital.

Max Drawdown: The largest peak-to-trough drop in your equity curve. This tells you the worst-case pain you would have felt.

Win Rate: The percentage of trades that were profitable.

Profit Factor: Gross Profit divided by Gross Loss. Anything above 1.0 is profitable.

Expectancy: The average return per trade.

Sharpe Ratio: Risk-adjusted return (volatility penalty). Higher is better.

Sortino Ratio: Like Sharpe, but only penalizes downside volatility.

Calmar Ratio: Return divided by Max Drawdown.

Kelly Criterion: A theoretical position-sizing formula. It is informational only; Chronos always uses 100% of equity.

Alpha, Beta, and Correlation: These are measured against the buy-and-hold of the same asset, not a market index.

4. The Probability Cloud (Full Edition)
The Probability Cloud uses a technique called Monte Carlo Permutation.
It takes your actual historical trades and shuffles their order 500 times. Because every path ends with the same final equity (just in a different sequence), the cloud shows you how much your drawdown changes depending on "when" the bad trades happen. It exposes sequence risk.

5. Exporting Reports
Tap EXPORT PDF REPORT to save a native PDF to your device. The PDF includes:

The exact strategy parameters.

The execution assumptions (fees, slippage, gap fills).

The Integrity data quality.

The full trade log.

The metrics.

You can use this report to compare results with other traders or to review your research later.

6. Troubleshooting
"Requested range needs ~X candles, cap is 1,000,000. Reduce years or use a higher timeframe." Chronos prevents you from downloading datasets that are too large for the phone's memory. Reduce the history or use a higher timeframe (e.g., 4h instead of 1m).

"Full Edition required." You need to purchase the Full Edition to use Keltner, ATR, or Monte Carlo.

Network errors: If you see "Rate limited," wait a minute. Google Play billing requires an internet connection to verify the entitlement.

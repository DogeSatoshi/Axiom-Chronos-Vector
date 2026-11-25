# Axiom Chronos Vector: Technical Specification & Architecture

## 1. Overview

Axiom Chronos Vector is a local-first, native Android application for financial market backtesting. Computation runs on the device; the only network use is downloading public market data. Its computational model follows established vectorized backtesting principles and is implemented natively in Kotlin, without a Python, NumPy or Pandas runtime.

## 2. Architectural paradigm

The application follows an **MVVM (Model-View-ViewModel)** pattern with a clear separation around the computational core.

- **UI layer:** declarative Jetpack Compose. Visualization, state and user interaction.
- **ViewModel layer:** coordinates data flow between providers and the engine and manages the lifecycle of a backtest session.
- **Core engine layer:**
  - **Data providers:** Binance (crypto) and Yahoo Finance (stocks and commodities) clients.
  - **Execution engine:** vectorized signal and trade simulation.
  - **Intelligence engine:** performance metrics and statistical analysis.
  - **Integrity layer:** data audit.

---

## 3. Core modules

### 3.1 Data acquisition

The data layer converts provider responses into OHLCV arrays.

- **Multi-year retrieval:** 1 to 10 years per request, within provider limits and a cap of 1,000,000 candles per request (Requests exceeding 1,000,000 candles are rejected to prevent memory exhaustion).
- **Timeframes:** 1m, 5m, 15m, 1h, 4h and 1d. Yahoo Finance does not offer a native 4h bar, so hourly bars are aggregated into trading-session 4h bars.
- **Closed candles only:** a still-forming candle is dropped.
- **Typed errors:** rate limit (honoring `Retry-After`), not found, restricted region, offline, server error and unreadable data are reported separately.
- **Session storage:** downloaded candles are held in memory during a session, are not written to disk, and are cleared when the market changes or the app closes.

### 3.2 Vectorized execution engine

The engine applies a strategy's signal array and the risk rules to the price arrays.

- **Positions:** long, and optional short positions (a user setting, off by default).
- **Risk rules:** stop loss, take profit and an optional ATR trailing stop (user settings in every market).
- **Costs:** a fee and a slippage charged on each side of every trade. The defaults are fixed per market and are not user-adjustable: crypto 0.10% + 0.05%, stocks 0.02% + 0.05%, commodities 0.05% + 0.05%.
- **Regime filter:** an optional 200-period EMA filter; no entries are taken until 200 bars of history exist.
- **Candle handling:** stops and targets are tested against each candle's high and low. Stops fill at the stop level, or at the candle's open if the candle gaps through it; take-profits fill at the target. If both are reachable in one candle, the stop is assumed to hit first.
- **Sizing:** each trade uses 100% of equity. No leverage, margin interest, borrow costs or funding rates are modeled.
- **Total-loss guard:** a position is closed when it has lost 98%.
- **End of data:** an open position is closed at the last close.

The engine models the selected assumptions consistently; it does not optimize results.

### 3.3 Quantitative intelligence

- **Performance:** total return, maximum drawdown (on closing equity), win rate, profit factor, expectancy and trade count.
- **Risk-adjusted:** Sharpe and Sortino ratios, annualized from the data's own bar frequency with no risk-free rate; Calmar ratio (CAGR divided by maximum drawdown); Kelly criterion (informational, shown as N/A when undefined).
- **Benchmark-relative:** alpha and beta from an ordinary least squares regression of strategy returns on the returns of buy-and-hold of the same asset (alpha annualized); Pearson correlation between the same two return series. No market index is used. These figures appear in the PDF report.
- **Monte Carlo analysis:** the strategy's trades are re-ordered at random without replacement, up to 500 times (fewer when a run has a very large number of trades; at least 20 trades are required). This exposes sequence risk: every re-ordering ends at the same final equity, so the analysis shows how much drawdown depends on trade order, not what will happen in the future.

### 3.4 Integrity engine

The integrity engine reports how complete the downloaded data is.

- **Continuous markets (crypto):** every missing candle between two timestamps is counted.
- **Stocks and futures:** bars are grouped by trading day (New York time; futures trading days start at 18:00 ET). Weekends, single-day closures and the daily futures break are not gaps. A day with fewer bars than the typical day counts its shortfall (early closes appear this way); three or more consecutive missing weekdays (two for daily bars) count as an outage. A single missing weekday cannot be told apart from a holiday and is not flagged. No holiday calendar is used.
- **Order and duplicates:** duplicate and out-of-order timestamps are counted on the data as received. The data providers' clients remove duplicates before the audit, so these counts are normally zero.
- **Volatility-aware weighting:** gaps that occur during volatile periods weigh more in the quality score.
- **Reliability rating:** combines data quality and trade count. Below 95% quality is *untrustworthy*; fewer than 30 trades is *low*; 30 to 99 trades, or under 99% quality, is *moderate*; 100 or more trades with at least 99% quality is *high*. It describes the evidence in a run, not the strategy's merit.

---

## 4. Logic flow: backtest lifecycle

1. **Ingestion:** the user selects an asset, timeframe and history length and fetches the data.
2. **Vectorization:** candles become arrays. Bars with zero volume (and, for stocks and commodities, flat bars) are dropped; the number of bars used is reported.
3. **Integrity audit:** the data as loaded is audited.
4. **Signal generation:** the selected strategy produces its signal array.
5. **Execution:** the engine applies trading, cost and risk assumptions to produce an equity curve and a trade list.
6. **Intelligence synthesis:** metrics, benchmark analysis and the Monte Carlo analysis are calculated.
7. **Session cleanup:** market data is discarded when the market changes or the app closes.

---

## 5. Strategies and markets

All ten strategies are available in every market: EMA Cross, MACD, RSI, Stochastic, Bollinger Breakout, ADX Trend, ROC, ATR Breakout, Money Flow Index and Keltner Channels. Keltner Channels and ATR Breakout are part of Full Edition. RSI, Stochastic and MFI strategies hold a position only while their condition is true.

Commodity data are continuous front-month futures series. Chronos does not model physical delivery, storage, weather, crop reports, mining disruptions or futures roll mechanics such as contango and backwardation, and the series are not roll-adjusted.

---

## 6. Performance characteristics

Chronos performs vector operations directly on the device. Run time depends on the device, the number of candles, the strategy and the number of trades. The Monte Carlo analysis is the most expensive step; it is memory-bounded and reduces the number of re-orderings, or is skipped, for very large trade counts. No benchmark figures are published for this version.

---

## 7. UI and visualization

- **HUD design system:** a Material-based visual system for dense quantitative information.
- **Charts:** equity curve, Monte Carlo percentile bands with sample paths, and a trade log (the last 20 trades on screen; the last 200 in the PDF).
- **Result transparency:** the loaded date range, the number of bars used, the cost assumptions and the reliability rating are shown with the results; the PDF report carries the full data-audit figures.

---

## 8. Technical stack

- **Language:** Kotlin 2.2
- **Concurrency:** Coroutines / StateFlow
- **UI:** Jetpack Compose, Material 3
- **Networking:** OkHttp 4.12
- **Charts:** MPAndroidChart 3.1
- **Billing:** Google Play Billing Library
- **Reporting:** native Android PDF generation, saved through the Android Storage Access Framework (the user chooses the location)
- **Minimum Android version:** Android 8.0 (API 26)

---

## 9. Privacy and security model

- **Local processing:** strategy execution and analysis happen on the user's device.
- **No accounts and no analytics:** the app contains no account system, analytics or advertising software.
- **Network use:** the app contacts the market-data providers (which see the user's IP address and the symbols, timeframes and date ranges requested) and Google Play (for purchases).
- **Stored data:** only a theme setting and a purchase flag with its last-checked time, in private app storage that is excluded from backups and device transfers. Candles are not stored.
- **Purchase check:** the app asks Google Play on the device whether Full Edition is owned, at launch and when it returns to the foreground. This is a client-side check; there is no server of ours involved, and a rooted device could bypass it.

External market-data providers are required to download history. Chronos does not guarantee that third-party data is available or complete; it audits what is received and reports what it can detect.

---

## 10. Scope and research position

Chronos is a **research and historical simulation instrument**, not a live trading system or a predictive oracle. It does not provide financial advice, guaranteed future performance or certainty about future market behavior. Results depend on the selected data, strategy parameters, execution assumptions and the market conditions of the historical period.

For a concise public overview, see [Readme.md](Readme.md).

---

*Compiled by Astra-Axiom Intelligence Systems.*

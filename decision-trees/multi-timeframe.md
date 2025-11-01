# Multi-timeframe Data Request Decision Tree

## 📊 Starting question: Do I need data from another timeframe?

```
┌─────────────────────────────────────────────────────┐
│   ⏰ Multi-timeframe data: request.security() guide  │
│   • Higher TF analysis • Lower TF precision • MTF confirm │
└─────────────────────────────────────────────────────┘
    │
    └─ 🎯 Why do I need multi-timeframe data?
        │
        ├─ Need higher timeframe trend direction
        │   └─ ➡️ **Trend filter**
        │
        ├─ Need lower timeframe precise entries
        │   └─ ➡️ **Precise timing**
        │
        ├─ Need multi-timeframe confirmation
        │   └─ ➡️ **Signal confirmation**
        │
        └─ Need indicator values from different TFs
            └─ ➡️ **Indicator comparison**
```

## 📈 Higher timeframe data

```
┌─ Need: fetch higher timeframe data (e.g., Daily/Weekly)
│
├─ 🔧 Basic syntax
│   │
│   └─ ```pine
│       data = request.security(
│           symbol,        // symbol
│           timeframe,     // target timeframe
│           expression,    // data to fetch
│           gaps,          // missing data handling
│           lookahead      // future data handling
│       )
│       ```
│
├─ 📊 Common scenarios
│   │
│   ├─ Trend direction
│   │   ```pine
│   │   // Use daily MA to determine higher trend
│   │   dailyMA200 = request.security(
│   │       syminfo.tickerid,
│   │       "1D",
│   │       ta.sma(close, 20)
│   │   )
│   │
│   │   // Only go long when higher trend is up
│   │   buySignal = close > ta.sma(close, 20) and close > dailyMA200
│   │   ```
│   │
│   ├─ Key price levels
│   │   ```pine
│   │   // Fetch daily support/resistance
│   │   dailyHigh = request.security(
│   │       syminfo.tickerid, "1D", high
│   │   )
│   │   dailyLow = request.security(
│   │       syminfo.tickerid, "1D", low
│   │   )
│   │
│   │   plot(dailyHigh, "Daily High", color.red)
│   │   plot(dailyLow, "Daily Low", color.green)
│   │   ```
│   │
│   └─ Indicator confirmation
│       ```pine
│       // Fetch daily RSI
│       dailyRSI = request.security(
│           syminfo.tickerid, "1D",
│           ta.rsi(close, 14)
│       )

│       // Multi-timeframe confirmation
│       m5Buy = ta.rsi(close, 14) < 30
│       d1TrendUp = dailyRSI > 50
│       finalBuy = m5Buy and d1TrendUp
│       ```
│
├─ ⚠️ Safe usage tips
│   │
│   ├─ Avoid future leaks
│   │   ```pine
│   │   // ❌ Danger: possible future leak
│   │   dailyClose = request.security(
│   │       syminfo.tickerid, "1D", close
│   │   )

│   │   // ✅ Safe: use offset
│   │   dailyClose = request.security(
│   │       syminfo.tickerid, "1D",
│   │       close[1],
│   │       lookahead=barmerge.lookahead_on
│   │   )
│   │   ```
│   │
│   ├─ Handle missing data
│   │   ```pine
│   │   // Handle missing data
│   │   weeklyData = request.security(
│   │       syminfo.tickerid, "W",
│   │       close,
│   │       gaps=barmerge.gaps_on,
│   │       lookahead=barmerge.lookahead_on
│   │   )
│   │
│   │   // Use nz() to handle na values
│   │   weeklyPrice = nz(weeklyData, close)
│   │   ```
│   │
│   └─ Caching optimization
│       ```pine
│       // Cache higher timeframe data
│       var float dailyMA = na
│       if ta.change(time("D"))
│           dailyMA := request.security(
│               syminfo.tickerid, "1D",
│               ta.sma(close, 20)[1]
│           )
│       ```
│
└─ 🎯 Best practices
    ├─ Use lookahead=barmerge.lookahead_on
    │   ```pine
    │   // Explicitly set lookahead
    │   data = request.security(..., lookahead=barmerge.lookahead_on)
    │   ```
    │
    ├─ Add offset for safety
    │   ```pine
    │   // Add [1] offset
    │   safeData = request.security(..., close[1])
    │   ```
    │
    └─ Validate data
        ```pine
        // Check data validity
        htfData = request.security(...)
        if not na(htfData)
            // use data
        ```
```

## 📉 Lower timeframe data

```
┌─ Need: fetch lower timeframe data (e.g., seconds/ticks)
│
├─ 📊 Use cases
│   │
│   ├─ Precise entry timing
│   │   ```pine
│   │   // On 1-minute chart, fetch 15-second data
│   │   sec15High = request.security(
│   │       syminfo.tickerid, "15S",
│   │       high,
│   │       gaps=barmerge.gaps_off,
│   │       lookahead=barmerge.lookahead_off
│   │   )

│   │   // Enter on breakout of 15-second high
│   │   if ta.crossover(close, sec15High)
│   │       strategy.entry("Long", strategy.long)
│   │   ```
│   │
│   ├─ Microstructure analysis
│   │   ```pine
│   │   // Get tick data (if available)
│   │   tickVolume = request.security(
│   │       syminfo.tickerid, "1S",
│   │       volume,
│   │       gaps=barmerge.gaps_off
│   │   )

│   │   // Detect short-term volume spikes
│   │   volumeSpike = tickVolume > ta.sma(tickVolume, 10) * 3
│   │   ```
│   │
│   └─ Scalping strategy
│       ```pine
│       // Quick in-out strategy
│       sec5Close = request.security(
│           syminfo.tickerid, "5S",
│           close[1]
│       )

│       // Fast MA crossover
│       sec5MA = ta.sma(sec5Close, 10)
│       if ta.crossover(sec5Close, sec5MA)
│           strategy.entry("Scalp", strategy.long)
│           strategy.exit("Exit", "Scalp", bars_after_entry=2)
│       ```
│
├─ ⚡ Performance optimization
│   │
│   ├─ Reduce request frequency
│   │   ```pine
│   │   // Request only when needed
│   │   var float lowTFData = na
│   │   if condition and barstate.isconfirmed
│   │       lowTFData := request.security(...)
│   │   ```
│   │
│   ├─ Use gaps=barmerge.gaps_off
│   │   ```pine
│   │   // Low timeframe data often has gaps
│   │   data = request.security(
│   │       ..., gaps=barmerge.gaps_off
│   │   )
│   │   ```
│   │
│   └─ Batch fetching
│       ```pine
│       // Request multiple values at once
│       [high5, low5, close5] = request.security(
│           syminfo.tickerid, "5S",
│           [high, low, close]
│       )
│       ```
│
└─ ⚠️ Notes
    ├─ Data availability
    │   - Not all timeframes are available
    │   - Depends on data source
    │
    ├─ Compute load
    │   - Low timeframe data is large
    │   - May impact performance
    │
    └─ Real-time considerations
        - Higher latency on very low TF
        - Consider real usage context
```

## 🔄 Multi-timeframe confirmation strategy

```
┌─ Need: multi-timeframe signal confirmation
│
├─ 📊 Strategy framework
│   └─ ```pine
│       //@version=6
│       strategy("Multi-timeframe Confirmation Strategy")
│
│       // Fetch multiple timeframe data
│       h1_trend = request.security("1", "60", ta.ema(close, 50))
│       h4_trend = request.security("1", "240", ta.ema(close, 50))
│       d1_trend = request.security("1", "D", ta.ema(close, 50))
│
│       // Multi-timeframe trend
│       trend_up = close > h1_trend and close > h4_trend and close > d1_trend
│       trend_down = close < h1_trend and close < h4_trend and close < d1_trend
│
│       // Current timeframe signal
│       m5_signal = ta.rsi(close, 14) < 30
│
│       // Confirmed by higher TF trend
│       if m5_signal and trend_up
│           strategy.entry("Long", strategy.long)
│       ```
│
├─ 🎯 Confirmation levels
│   │
│   ├─ Strong confirmation (all TF align)
│   │   ```pine
│       // All three TF bullish
│       strongBuy = m5_rsi < 30 and h1_rsi < 30 and h4_rsi < 30
│       ```
│   │
│   ├─ Medium confirmation (majority agree)
│   │   ```pine
│       // Any two TF bullish
│       mediumBuy = (m5_rsi < 30 and h1_rsi < 30) or
│                   (m5_rsi < 30 and h4_rsi < 30) or
│                   (h1_rsi < 30 and h4_rsi < 30)
│       ```
│   │
│   └─ Weak confirmation (higher TF leads)
│       ```pine
│       // As long as higher TF is bullish
│       weakBuy = d1_trend_up and m5_signal
│       ```
│
├─ 📈 Dynamic timeframe selection
│   └─ ```pine
│       // Choose timeframe based on volatility
│       atr = ta.atr(14)
│       volatility_level = atr / close * 100

│       // Use longer period for high volatility
│       htf_period = volatility_level > 2 ? "4H" : "1H"

│       htf_data = request.security(
│           syminfo.tickerid,
│           htf_period,
│           ta.sma(close, 20)
│       )
│       ```
│
└─ 💡 Practical tips
    ├─ Use a scoring system
    │   ```pine
    │   // Multi-timeframe scoring
    │   score = 0
    │   if m5_condition => score += 1
    │   if h1_condition => score += 2
    │   if h4_condition => score += 3
    │   if d1_condition => score += 4

    │   // Decide based on score
    │   if score >= 7
    │       strategy.entry(...)
    │   ```
    │
    ├─ Timeframe filter
    │   ```pine
    │   // Adjust strategy based on timeframe
    │   currentTF = timeframe.period
    │   isHigherTF = currentTF in ["60", "240", "D"]
    │
    │   if isHigherTF
    │       // Use looser conditions
    │       threshold = 25
    │   else
    │       // Use stricter conditions
    │       threshold = 20
    │   ```
    │
    └─ Progressive confirmation
        ```pine
        // Step-by-step confirmation
        if m5_signal
            // Level 1: Basic signal
            plotshape(1, "L1", style=shape.circle)

        if m5_signal and h1_confirm
            // Level 2: 1H confirmation
            plotshape(2, "L2", style=shape.square)

        if m5_signal and h1_confirm and h4_confirm
            // Level 3: 4H confirmation
            plotshape(3, "L3", style=shape.diamond)
        ```
```

## 📊 request.security() parameters explained

```
┌─ Parameter details
│
├─ symbol (instrument)
│   │
│   ├─ Use current instrument
│   │   ```pine
│   │   syminfo.tickerid
│   │   ```
│   │
│   ├─ Specify another instrument
│   │   ```pine
│   │   "NASDAQ:AAPL"
│   │   "BINANCE:BTCUSDT"
│   │   ```
│   │
│   └─ Dynamic instrument
│       ```pine
│       // Choose instrument based on input
│       sym = input.symbol("NASDAQ:TSLA")
│       data = request.security(sym, "D", close)
│       ```
│
├─ timeframe (time frame)
│   │
│   ├─ Predefined timeframes
│   │   ```pine
│   │   "1", "5", "15", "30", "60", "240", "D", "W", "M"
│   │   ```
│   │
│   ├─ Custom timeframes
│   │   ```pine
│   │   "120"     // 2 hours
│   │   "720"     // 12 hours
│   │   "3D"      // 3 days
│   │   "2W"      // 2 weeks
│   │   ```
│   │
│   └─ Relative timeframe
│       ```pine
│       // Multiple of current timeframe
│       multiplier = 4
│       htfTimeframe = str.tostring(timeframe.multiplier * multiplier)
│       ```
│
├─ expression (data expression)
│   │
│   ├─ Basic data
│   │   ```pine
│   │   close, high, low, open, volume
│   │   ```
│   │
│   ├─ Indicator calculations
│   │   ```pine
│   │   ta.sma(close, 20)
│   │   ta.rsi(close, 14)
│   │   ta.macd(close, 12, 26, 9)
│   │   ```
│   │
│   ├─ Composite expressions
│   │   ```pine
│   │   ta.sma(close, 20) > ta.sma(close, 50)
│   │   ta.rsi(close, 14) > 50
│   │   ```
│   │
│   └─ Multiple return values
│       ```pine
│       [ma20, ma50] = request.security(
│           syminfo.tickerid, "D",
│           [ta.sma(close, 20), ta.sma(close, 50)]
│       )
│       ```
│
├─ gaps (missing data handling)
│   │
│   ├─ barmerge.gaps_on
│   │   - Returns na values
│   │   - Suitable when you need to know data is missing
│   │
│   ├─ barmerge.gaps_off
│   │   - Returns the last valid value
│   │   - Suitable for continuous data
│   │
│   └─ Selection advice
│       ```pine
│       // Use gaps_on for higher timeframe data
│       dailyData = request.security(..., gaps=barmerge.gaps_on)

│       // Use gaps_off for lower timeframe data
│       tickData = request.security(..., gaps=barmerge.gaps_off)
│       ```
│
└─ lookahead (future data handling)
    │
    ├─ barmerge.lookahead_on
    │   - Allows accessing future data (data from the current unfinished bar)
    │   - May cause repainting
    │
    ├─ barmerge.lookahead_off
    │   - Only access confirmed data
    │   - Avoids repainting
    │
    └─ Safety advice
        ```pine
        // In most cases, use lookahead_on + offset
        data = request.security(
            syminfo.tickerid, "D",
            close[1],
            lookahead=barmerge.lookahead_on
        )

        // When you must absolutely avoid repainting
        data = request.security(
            syminfo.tickerid, "D",
            close,
            lookahead=barmerge.lookahead_off
        )
        ```
```

## ⚠️ Common issues and solutions

### Issue 1: Repainting
```pine
// ❌ Causes repainting
dailyClose = request.security(syminfo.tickerid, "D", close)

// ✅ Solution 1: use offset
dailyClose = request.security(syminfo.tickerid, "D", close[1])

// ✅ Solution 2: use lookahead_off
dailyClose = request.security(
    syminfo.tickerid, "D",
    close,
    lookahead=barmerge.lookahead_off
)
```

### Issue 2: Performance
```pine
// ❌ Request on every bar
if condition
    data = request.security(...)

// ✅ Cache data
var float cachedData = na
if ta.change(time("D"))  // Update once per day
    cachedData := request.security(...)
```

### Issue 3: Data discontinuity
```pine
// ❌ Direct use may lead to errors
weeklyData = request.security(...)

// ✅ Handle na values
weeklyData = request.security(..., gaps=barmerge.gaps_on)
cleanData = nz(weeklyData, defaultValue)
```

### Issue 4: Timeframe mismatch
```pine
// ❌ May fail on non-standard timeframes
data = request.security(syminfo.tickerid, "45", close)

// ✅ Use a standard timeframe
data = request.security(syminfo.tickerid, "60", close)
```

## 📋 Best practices summary

1. **Always consider repainting risk**
   - Prefer using offset [1]
   - Explicitly set the lookahead parameter

2. **Optimize performance**
   - Cache higher timeframe data
   - Reduce unnecessary requests
   - Batch fetch multiple values

3. **Handle data quality**
   - Check na values
   - Set appropriate gaps parameter
   - Validate data validity

4. **Design sensible strategies**
   - Don't over-rely on higher timeframes
   - Consider data latency
   - Balance complexity and effectiveness

5. **Testing and validation**
   - Test on historical data
   - Observe real-time performance
   - Mind differences across markets

## 💡 Quick reference table

| Scenario | Recommended parameter settings |
|------|-------------|
| Trend filter | `close[1]`, `lookahead_on`, `gaps_on` |
| Precise entry | `close`, `lookahead_off`, `gaps_off` |
| Indicator comparison | `ta.indicator()[1]`, `lookahead_on` |
| Data caching | Use `var` variable to store |
| Multiple symbols | Specify full symbol path |

Remember: Multi-timeframe data is powerful but requires caution—always consider repainting risk and performance impact.
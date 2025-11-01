# Repainting Errors Anti-Patterns

Repainting is one of the most common and hardest-to-detect issues in Pine Script. The following are error patterns that most easily cause repainting.

## 1. request.security() Future Leak

### ❌ Incorrect example: using live data directly
```pine
//@version=6
indicator("Error: Future leak", overlay=true)

// ❌ Dangerous: will fetch unfinished daily data
dailyClose = request.security(syminfo.tickerid, "D", close)

// Display tomorrow's daily data on the current bar
plot(dailyClose, "Daily close", color.red, 2)

// Signals based on future data
buySignal = close > dailyClose
plotshape(buySignal, style=shape.triangleup, location=location.belowbar)
```

### 🚨 Problem explanation
- `request.security()` by default returns live data for the current timeframe
- Daily data changes until the bar is complete, causing "future data" to leak into the present
- Signals on live bars keep changing; backtests look perfect but are not tradable

### ✅ Correct approach: use an offset
```pine
//@version=6
indicator("Correct: Avoid future leak", overlay=true)

// ✅ Safe: use confirmed daily data
dailyClose = request.security(
    syminfo.tickerid,
    "D",
    close[1],  // Use the previous confirmed daily bar
    lookahead=barmerge.lookahead_on  // Explicitly set lookahead
)

// Show only confirmed data
plot(dailyClose, "Daily close", color.blue, 2)

// Signals based on historical data
buySignal = close > dailyClose
plotshape(buySignal, style=shape.triangleup, location=location.belowbar)
```

## 2. Signal Repainting on Live Bars

### ❌ Incorrect example: generating signals from live data
```pine
//@version=6
indicator("Error: Live signal repainting")

// ❌ Using live close values (changes on live bars)
rsi = ta.rsi(close, 14)
oversoldSignal = ta.crossunder(rsi, 30)

// The signal repeatedly appears/disappears on live bars
plotshape(oversoldSignal, "Oversold signal",
          style=shape.labelup, location=location.belowbar)

// Entry labels based on live signals
if oversoldSignal
    label.new(bar_index, low, "Buy", color.green)
```

### 🚨 Problem explanation
- `close` changes continuously on live bars
- RSI values change accordingly, causing signals to trigger repeatedly
- Labels appear and disappear with the signal, causing confusion

### ✅ Correct approach: wait for bar confirmation
```pine
//@version=6
indicator("Correct: Confirmed signal")

// ✅ Use barstate.isconfirmed to ensure bar completion
rsi = ta.rsi(close, 14)
oversoldSignal = ta.crossunder(rsi[1], 30) and barstate.isconfirmed

// Signal only triggers once at bar close
plotshape(oversoldSignal, "Oversold signal",
          style=shape.labelup, location=location.belowbar)

// Use var to avoid label repainting
var label buyLabel = na
if oversoldSignal
    if na(buyLabel)
        buyLabel := label.new(bar_index, low, "Buy", color.green)
else
    if not na(buyLabel)
        label.delete(buyLabel)
        buyLabel := na
```

## 3. Dynamic Support/Resistance Repainting

### ❌ Incorrect example: dynamically adjusting historical levels
```pine
//@version=6
indicator("Error: Dynamic support/resistance", overlay=true)

// ❌ Using pivot points while allowing historical adjustment
leftLen = input.int(10, "Left length")
rightLen = input.int(10, "Right length")

// pivot.high allows future data to affect historical pivots
pivothigh = ta.pivothigh(high, leftLen, rightLen)

// Historical pivots move as new bars arrive
plot(pivothigh, "Resistance", color.red)
```

### 🚨 Problem explanation
- `ta.pivothigh` by default allows future data to adjust historical pivot points
- When new bars appear, past pivot points can change position
- Backtests diverge from real trading

### ✅ Correct approach: fix historical pivots
```pine
//@version=6
indicator("Correct: Fixed support/resistance", overlay=true)

// ✅ Use var to fix historical pivots
var float resistanceLevel = na
var float supportLevel = na
var int lastPivotBar = na

// Detect new pivot points (confirmed only)
ph = ta.pivothigh(high, 10, 10)
pl = ta.pivotlow(low, 10, 10)

// Update only after confirmation (do not change history)
if not na(ph)
    resistanceLevel := ph
    lastPivotBar := bar_index

if not na(pl)
    supportLevel := pl
    lastPivotBar := bar_index

// Draw fixed horizontal lines
plot(resistanceLevel, "Resistance", color.red, style=plot.style_linebr)
plot(supportLevel, "Support", color.green, style=plot.style_linebr)
```

## 4. Using Future Data in Calculations

### ❌ Incorrect example: using future values on live bars
```pine
//@version=6
indicator("Error: Future-based calculation")

// ❌ Using high/low on live bars
atr = ta.atr(14)
upperBand = high + atr
lowerBand = low - atr

// High/low change on live bars
plot(upperBand, "Upper band", color.red)
plot(lowerBand, "Lower band", color.green)

// Breakout signal based on future values
breakout = ta.crossover(close, upperBand)
plotshape(breakout, "Breakout", style=shape.triangleup)
```

### 🚨 Problem explanation
- `high` and `low` keep updating on live bars
- Channels drawn from these values change in real time
- Breakout signals may trigger before close and then disappear

### ✅ Correct approach: use confirmed values
```pine
//@version=6
indicator("Correct: Confirmed-value calculation")

// ✅ Use prior bar’s confirmed values
prevHigh = high[1]
prevLow = low[1]
prevAtr = ta.atr(14)[1]

// Compute fixed channel
upperBand = prevHigh + prevAtr
lowerBand = prevLow - prevAtr

// Channel values are fixed on the current bar
plot(upperBand, "Upper band", color.red)
plot(lowerBand, "Lower band", color.green)

// Breakout signal based on fixed values
breakout = ta.crossover(close, upperBand)
plotshape(breakout, "Breakout", style=shape.triangleup)
```

## 5. Multi-Timeframe Repainting

### ❌ Incorrect example: mixing timeframes leads to inconsistency
```pine
//@version=6
indicator("Error: MTF repainting", overlay=true)

// ❌ Fetching different timeframe data without synchronization
m5_close = request.security(syminfo.tickerid, "5", close)
h1_close = request.security(syminfo.tickerid, "60", close)

// Different timeframes update at different frequencies
// Leads to indicator inconsistency over time
alignment = m5_close > h1_close
bgcolor(alignment ? color.green : color.red, transp=90)

// Trading based on alignment signal
plotshape(alignment, "Alignment signal", style=shape.circle)
```

### 🚨 Problem explanation
- Different timeframes update at different frequencies
- 5-minute data updates every 5 minutes; 1-hour data updates hourly
- Background color becomes na or incorrect most of the time

### ✅ Correct approach: synchronize updates
```pine
//@version=6
indicator("Correct: MTF synchronization", overlay=true)

// ✅ Cache higher timeframe data
var float h1_close = na
var int lastHour = na

// Update only on hour change
currentHour = hour(time)
if currentHour != lastHour
    h1_close := request.security(
        syminfo.tickerid,
        "60",
        close[1],
        lookahead=barmerge.lookahead_on
    )
    lastHour := currentHour

// Use 5-minute data from current timeframe
m5_close = close

// Both values are confirmed
alignment = m5_close > h1_close
bgcolor(not na(alignment) ?
        (alignment ? color.green : color.red) : na,
        transp=90)

plotshape(alignment, "Alignment signal", style=shape.circle)
```

## 6. Circular References Causing Repainting

### ❌ Incorrect example: circular dependency
```pine
//@version=6
indicator("Error: Circular reference")

// ❌ value depends on its own historical values
// But on live bars it updates recursively
var float value = 0
value := ta.sma(value, 10) + close * 0.1

// On live bars, value keeps repainting
plot(value, "Cyclic value", color.red)
```

### 🚨 Problem explanation
- `value` depends on its own smoothed value
- On live bars, each update affects subsequent calculations
- Causes unbounded recursive repainting

### ✅ Correct approach: explicit computation chain
```pine
//@version=6
indicator("Correct: Explicit computation")

// ✅ Use an independent computation chain
rawValue = close * 0.1
var float[] rawHistory = array.new<float>(10, 0)

// Update only on bar confirmation
if barstate.isconfirmed
    array.unshift(rawHistory, rawValue)
    if array.size(rawHistory) > 10
        array.pop(rawHistory)

// Compute based on historical data
smoothValue = array.avg(rawHistory)
plot(smoothValue, "Smoothed value", color.blue)
```

## 7. timenow Misuse

### ❌ Incorrect example: using current time
```pine
//@version=6
indicator("Error: timenow usage")

// ❌ timenow changes continuously on live bars
isSessionEnd = hour(timenow) >= 15 and minute(timenow) >= 30

// The condition changes with real time
bgcolor(isSessionEnd ? color.red : na)

// Time-based signal
if isSessionEnd
    label.new(bar_index, high, "Session close time", color.red)
```

### 🚨 Problem explanation
- `timenow` is the script's real-time clock
- It constantly changes on live bars
- Judgments on historical bars also change as time passes

### ✅ Correct approach: use bar time
```pine
//@version=6
indicator("Correct: Bar time")

// ✅ Use the bar's own time
isSessionEnd = hour(time) >= 15 and minute(time) >= 30

// The judgment is based on bar time and is fixed
bgcolor(isSessionEnd ? color.green : na)

// Show real-time info only on the last bar
if barstate.islast
    label.new(bar_index, high,
              "Current time: " + str.format("{0,time,HH:mm}", timenow),
              color.blue)
```

## Checklist for Repainting

1. **Using request.security() without an offset?**
2. **Using close/high/low on live bars?**
3. **Waiting for barstate.isconfirmed?**
4. **Using timenow for historical judgments?**
5. **Variables with circular dependencies?**
6. **Mixing data with different update frequencies?**

## Golden Rules to Avoid Repainting

1. **Always use offsets**: `value[1]` is your friend
2. **Wait for confirmation**: `barstate.isconfirmed` is the safety lock
3. **Fix history**: historical data should not change
4. **Avoid timenow**: unless used only on the last bar
5. **Be explicit about lookahead**: understand request.security behavior

Remember: **If backtest results look too good to be true, repainting likely exists!**
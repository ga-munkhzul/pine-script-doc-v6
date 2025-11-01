# 重绘错误反例

重绘是 Pine Script 中最常见且最难发现的问题之一。以下是最容易导致重绘的错误模式。

## 1. request.security() 未来泄漏

### ❌ 错误示例：直接使用实时数据
```pine
//@version=6
indicator("错误：未来泄漏", overlay=true)

// ❌ 危险：会获取未完成的日线数据
dailyClose = request.security(syminfo.tickerid, "D", close)

// 在当前K线显示明天的日线数据
plot(dailyClose, "日线收盘", color.red, 2)

// 基于未来数据的信号
buySignal = close > dailyClose
plotshape(buySignal, style=shape.triangleup, location=location.belowbar)
```

### 🚨 问题说明
- `request.security()` 默认会返回当前周期的实时数据
- 日线在完成前会不断变化，导致"未来数据"泄漏到当前
- 信号在实时K线上会不断变化，历史表现看起来完美但实际不可行

### ✅ 正确做法：使用偏移
```pine
//@version=6
indicator("正确：避免未来泄漏", overlay=true)

// ✅ 安全：使用已确认的日线数据
dailyClose = request.security(
    syminfo.tickerid,
    "D",
    close[1],  // 使用前一根已确认的日线
    lookahead=barmerge.lookahead_on  // 明确设置lookahead
)

// 仅显示已确认的数据
plot(dailyClose, "日线收盘", color.blue, 2)

// 基于历史数据的信号
buySignal = close > dailyClose
plotshape(buySignal, style=shape.triangleup, location=location.belowbar)
```

## 2. 实时K线上的信号重绘

### ❌ 错误示例：使用实时数据生成信号
```pine
//@version=6
indicator("错误：实时信号重绘")

// ❌ 使用实时close值（会在实时K线上变化）
rsi = ta.rsi(close, 14)
oversoldSignal = ta.crossunder(rsi, 30)

// 信号会在实时K线上反复出现和消失
plotshape(oversoldSignal, "超卖信号",
          style=shape.labelup, location=location.belowbar)

// 基于实时信号的入场标记
if oversoldSignal
    label.new(bar_index, low, "买入", color.green)
```

### 🚨 问题说明
- `close` 在实时K线上会不断变化
- RSI值会随之变化，导致信号反复触发
- 标签会随着信号出现和消失，造成混乱

### ✅ 正确做法：等待K线确认
```pine
//@version=6
indicator("正确：确认信号")

// ✅ 使用barstate.isconfirmed确保K线完成
rsi = ta.rsi(close, 14)
oversoldSignal = ta.crossunder(rsi[1], 30) and barstate.isconfirmed

// 信号只会在K线收盘时触发一次
plotshape(oversoldSignal, "超卖信号",
          style=shape.labelup, location=location.belowbar)

// 使用var避免标签重绘
var label buyLabel = na
if oversoldSignal
    if na(buyLabel)
        buyLabel := label.new(bar_index, low, "买入", color.green)
else
    if not na(buyLabel)
        label.delete(buyLabel)
        buyLabel := na
```

## 3. Dynamic support/resistance repainting

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

## 4. Using future data in calculations

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

## 5. Multi-timeframe repainting

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

## 6. Circular references causing repainting

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

## 7. timenow misuse

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
- 历史K线上的判断也会随着时间推移而改变

### ✅ 正确做法：使用K线时间
```pine
//@version=6
indicator("正确：K线时间")

// ✅ 使用K线自己的时间
isSessionEnd = hour(time) >= 15 and minute(time) >= 30

// 判断基于K线时间，固定不变
bgcolor(isSessionEnd ? color.green : na)

// 只在最后一根K线显示实时信息
if barstate.islast
    label.new(bar_index, high,
              "当前时间: " + str.format("{0,time,HH:mm}", timenow),
              color.blue)
```

## 检查重绘的清单

1. **是否使用 request.security() 而没有偏移？**
2. **是否在实时K线上使用 close/high/low？**
3. **是否等待 barstate.isconfirmed？**
4. **是否使用 timenow 进行历史判断？**
5. **是否有循环依赖的变量？**
6. **是否混合不同更新频率的数据？**

## 避免重绘的黄金法则

1. **总是使用偏移**：`value[1]` 是你的朋友
2. **等待确认**：`barstate.isconfirmed` 是安全锁
3. **固定历史**：历史数据不应该改变
4. **避免 timenow**：除非只用于最后一根K线
5. **明确 lookahead**：清楚了解 request.security 的行为

记住：**如果回测结果看起来太美好，很可能存在重绘问题！**
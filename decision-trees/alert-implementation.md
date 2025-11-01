# Alert Implementation Decision Tree

## 🔔 Starting question: What type of alert do I need to implement?

```
┌─────────────────────────────────────────────────────┐
│   ⚠️ Pine Script alert type selection                │
│   • alertcondition() - UI condition alert            │
│   • alert() - dynamic message alert                  │
│   • Strategy order-fill alerts                       │
└─────────────────────────────────────────────────────┘
    │
    └─ 📋 Alert requirement?
        │
        ├─ Simple condition trigger (user selects in UI)
        │   └─ ➡️ **alertcondition()**
        │
        ├─ Complex dynamic messages
        │   └─ ➡️ **alert()**
        │
        ├─ Strategy trade signals
        │   └─ ➡️ **Order-fill alerts**
        │
        └─ Multiple alert types combined
            └─ ➡️ **Mixed approach**
```

## 📊 alertcondition() - UI condition alerts

```
┌─ You chose: alertcondition()
│
├─ ✅ Applicable scenarios
│   │
│   ├─ Simple buy/sell signals
│   │   ```pine
│   │   // Crossover signal
│   │   [macdLine, signalLine] = ta.macd(close, 12, 26, 9)
│   │   bullishCross = ta.crossover(macdLine, signalLine)
│   │   bearishCross = ta.crossunder(macdLine, signalLine)
│   │
│   │   // Create alert conditions
│   │   alertcondition(bullishCross, "MACD Bullish Cross")
│   │   alertcondition(bearishCross, "MACD Bearish Cross")
│   │   ```
│   │
│   ├─ Price level breakouts
│   │   ```pine
│   │   // Support/resistance breakout
│   │   resistance = ta.highest(high, 20)
│   │   support = ta.lowest(low, 20)
│   │
│   │   alertcondition(ta.crossover(close, resistance),
│   │                  "Break resistance")
│   │   alertcondition(ta.crossunder(close, support),
│   │                  "Break support")
│   │   ```
│   │
│   └─ Indicator threshold triggers
│       ```pine
│       // RSI overbought/oversold
│       rsi = ta.rsi(close, 14)
│       alertcondition(rsi > 70, "RSI Overbought")
│       alertcondition(rsi < 30, "RSI Oversold")
│       ```
│
├─ 🔧 Basic syntax
│   │
│   ├─ Simple form
│   │   ```pine
│   │   alertcondition(condition, title)
│   │   ```
│   │
│   ├─ With message
│   │   ```pine
│   │   alertcondition(condition, title, message)
│   │   ```
│   │
│   └─ Full parameters
│       ```pine
│       alertcondition(
│           condition,      // Condition expression
│           title,          // Title shown in the Alert dialog
│           message         // Alert message template
│       )
│       ```
│
├─ 📝 Message templates
│   │
│   ├─ Use predefined variables
│   │   ```pine
│   │   alertcondition(
│   │       buySignal,
│   │       "Buy signal",
│   │       "Buy {{ticker}} at {{close}}"
│   │   )
│   │
│   │   // Available variables:
│   │   // {{exchange}}, {{ticker}}, {{interval}}
│   │   // {{close}}, {{high}}, {{low}}, {{open}}
│   │   // {{time}}, {{timenow}}
│   │   ```
│   │
│   ├─ Custom variables
│   │   ```pine
│   │   // ❌ Wrong: cannot use custom variables
│   │   alertcondition(signal, "Signal", "Price: {{myPrice}}")
│   │
│   │   // ✅ Correct: use built-in variables
│   │   alertcondition(signal, "Signal", "Price: {{close}}")
│   │   ```
│   │
│   └─ Multi-language support
│       ```pine
│       // Chinese message
│       alertcondition(buySignal, "Buy",
│                      "Buy {{ticker}} at {{close}}")
│       ```
│
├─ ⚠️ Limitations and notes
│   │
│   ├─ Indicators only
│   │   ❌ Cannot be used in strategies
│   │
│   ├─ Messages are static
│   │   ❌ Cannot include dynamically computed values
│   │   ✅ Only predefined variables allowed
│   │
│   ├─ Count limits
│   │   - Up to 20 alertcondition() calls
│   │   - Counts towards script plot count
│   │
│   └─ Trigger frequency
│       - Triggers only on realtime bars
│       - Requires user activation in UI
│
└─ 💡 Best practices
    ├─ Clear titles
    │   ```pine
    │   // ✅ Good title
    │   alertcondition(signal, "MACD Golden Cross (12,26,9)")
    │
    │   // ❌ Vague title
    │   alertcondition(signal, "Signal")
    │   ```
    │
    ├─ Descriptive messages
    │   ```pine
    │   alertcondition(
    │       crossUp,
    │       "MA20 crosses above MA50",
    │       "Bullish MA crossover: {{ticker}} {{interval}}"
    │   )
    │   ```
    │
    └─ Combined conditions
        ```pine
        // Multiple confirmations
        strongBuy = rsi < 30 and close > ta.sma(close, 50)
        alertcondition(strongBuy, "Strong buy signal",
                      "RSI oversold and price above MA")
        ```
```

## 🚨 alert() - Dynamic message alerts

```
┌─ You chose: alert()
│
├─ ✅ Applicable scenarios
│   │
│   ├─ Dynamic price alerts
│   │   ```pine
│   │   // Custom price calculation
│   │   entryPrice = ta.lowest(low, 5) * 1.02
│   │   if close > entryPrice and barstate.isconfirmed
│   │       alert(
│   │           "Broke entry price: " + str.tostring(close) +
│   │           " Target: " + str.tostring(close * 1.05),
│   │           alert.freq_once_per_bar_close
│   │       )
│   │   ```
│   │
│   ├─ Composite multi-indicator signal
│   │   ```pine
│   │   // Composite condition alert
│   │   rsi = ta.rsi(close, 14)
│   │   macd = ta.macd(close)[0]
│   │   volumeSpike = volume > ta.sma(volume, 20) * 2
│   │
│   │   if rsi < 30 and macd > 0 and volumeSpike
│   │       alertMsg = "Multi-confirmation buy signal:\n" +
│   │                    "RSI: " + str.tostring(rsi, "#.##") + "\n" +
│   │                    "MACD: " + str.tostring(macd, "#.####") + "\n" +
│   │                    "Volume: " + str.tostring(volume)
│   │       alert(alertMsg, alert.freq_once_per_bar)
│   │   ```
│   │
│   ├─ Realtime monitoring alerts
│   │   ```pine
│   │   // Realtime price change alert
│   │   priceChange = math.abs(close - close[1]) / close[1] * 100
│   │   if priceChange > 5
│   │       alert("Sharp price change: " +
│   │              str.tostring(priceChange, "#.##") + "%",
│   │              alert.freq_all)
│   │   ```
│   │
│   └─ Strategy state alerts
│       ```pine
│       // Strategy state changes
│       var bool inPosition = false
│       if strategy.position_size != 0 and not inPosition
│           inPosition := true
│           alert("Enter position: " + str.tostring(strategy.position_avg_price))
│       else if strategy.position_size == 0 and inPosition
│           inPosition := false
│           alert("Exit all positions")
│       ```
│
├─ 🔧 Frequency control
│   │
│   ├─ alert.freq_once_per_bar
│   │   ```pine
│   │   // Trigger only once per bar
│   │   if condition
│   │       alert("Signal", alert.freq_once_per_bar)
│   │   ```
│   │
│   ├─ alert.freq_once_per_bar_close
│   │   ```pine
│   │   // Trigger only at bar close (avoid repainting)
│   │   if condition and barstate.isconfirmed
│   │       alert("Confirmed signal", alert.freq_once_per_bar_close)
│   │   ```
│   │
│   └─ alert.freq_all
│       ```pine
│       // Trigger every time the condition is met (use carefully)
│       if realtimeCondition
│           alert("Realtime update", alert.freq_all)
│       ```
│
├─ 📊 Advanced features
│   │
│   ├─ Combine conditions
│   │   ```pine
│   │   // Multiple-condition alert
│   │   if condition1 and condition2 and barstate.isconfirmed
│   │       alert("Combined conditions met", alert.freq_once_per_bar_close)
│   │   ```
│   │
│   ├─ Counting and rate limiting
│   │   ```pine
│   │   // Rate-limit alerts
│   │   var int lastAlertBar = 0
│   │   minBarsBetween = 20
│   │
│   │   if condition and bar_index - lastAlertBar > minBarsBetween
│   │       alert("Signal", alert.freq_once_per_bar)
│   │       lastAlertBar := bar_index
│   │   ```
│   │
│   └─ Formatted messages
│       ```pine
│       // Rich text formatting
│       alertMsg = "🔥 " + title + "\n" +
│                  "Price: " + str.tostring(close, "#.##") + "\n" +
│                  "Time: " + str.format("{0,date,yyyy-MM-dd HH:mm}", time)
│       alert(alertMsg)
│       ```
│
├─ ⚠️ Notes
│   │
│   ├─ Realtime limitations
│   │   - alert() runs only on realtime bars
│   │   - Historical bars won't trigger alerts
│   │
│   ├─ Performance considerations
│   │   - Avoid too many alert calls
│   │   - Set appropriate frequency
│   │
│   └─ Message length
│       - Alert message length is limited
│       - Keep messages concise
│
└─ 💡 Best practices
    ├─ Use barstate.isconfirmed
    │   ```pine
    │   // Avoid false positives from repainting
    │   if signal and barstate.isconfirmed
    │       alert("Confirmed signal", alert.freq_once_per_bar_close)
    │   ```
    │
    ├─ Clear message format
    │   ```pine
    │   // Structured message
    │   alert("[Buy signal]\n" +
    │        "Symbol: {{ticker}}\n" +
    │        "Price: " + str.tostring(close) + "\n" +
    │        "Indicator: " + str.tostring(rsi, "#.##"))
    │   ```
    │
    └─ Error handling
        ```pine
        // Validate data before sending alerts
        if not na(close) and not na(volume) and condition
            alert("Valid signal", alert.freq_once_per_bar)
        ```
```

## 📈 Strategy order-fill alerts

```
┌─ You chose: strategy order alerts
│
├─ ✅ Automatic order alerts
│   │
│   └─ Auto-triggered on order execution
│       ```pine
│       //@version=6
│       strategy("Trading strategy", overlay=true)
│
│       // Auto-trigger alert when order is filled
│       if ta.crossover(close, ta.sma(close, 20))
│           strategy.entry("Long", strategy.long,
│                         alert_message="Buy order executed")
│
│       if ta.crossunder(close, ta.sma(close, 20))
│           strategy.entry("Short", strategy.short,
│                         alert_message="Sell order executed")
│       ```
│
├─ 🔧 Custom order messages
│   │
│   ├─ Entry orders
│   │   ```pine
│       strategy.entry(
│           "Long",
│           strategy.long,
│           qty=0.1,
│           limit=entryPrice,
│           stop=stopLoss,
│           alert_message=
│               "Buy executed - Price: {{strategy.order.price}} " +
│               "Size: {{strategy.order.size}} " +
│               "Type: {{strategy.order.comment}}"
│       )
│       ```
│   │
│   ├─ Exit orders
│   │   ```pine
│       strategy.exit(
│           "Exit Long",
│           "Long",
│           profit=100,
│           loss=50,
│           alert_message=
│               "Exit long - P/L: {{strategy.order.comment}}"
│       )
│       ```
│   │
│   └─ Custom fields
│       ```pine
│       // Use comment to pass extra info
│       strategy.entry(
│           "Trade",
│           strategy.long,
│           comment="RSI:" + str.tostring(rsi, "#"),
│           alert_message="{{strategy.order.comment}}"
│       )
│       ```
│
├─ 📊 Available template variables
│   │
│   ├─ Order info
│   │   - `{{strategy.order.price}}` - Execution price
│   │   - `{{strategy.order.size}}` - Order size
│   │   - `{{strategy.order.comment}}` - Order comment
│   │   - `{{strategy.order.id}}` - Order ID
│   │
│   ├─ Strategy info
│   │   - `{{strategy.position_size}}` - Current position size
│   │   - `{{strategy.position_avg_price}}` - Average price
│   │   - `{{strategy.closed_trades}}` - Closed trades count
│   │
│   ├─ Performance metrics
│   │   - `{{strategy.gross_profit}}` - Gross profit
│   │   - `{{strategy.gross_loss}}` - Gross loss
│   │   - `{{strategy.netprofit}}` - Net profit
│   │
│   └─ Market info
│       - `{{ticker}}` - Symbol
│       - `{{close}}` - Close
│       - `{{time}}` - Time
│
└─ 💡 Advanced usage
    ├─ Conditional alert messages
    │   ```pine
    │   // Different messages depending on context
    │   alertMsg = profit > 0 ?
    │                "Profit {{strategy.order.comment}}" :
    │                "Loss {{strategy.order.comment}}"
    │   strategy.exit(..., alert_message=alertMsg)
    │   ```
    │
    ├─ Dynamic calculations
    │   ```pine
    │   // Include computed values
    │   riskReward = reward / risk
    │   strategy.entry(...,
    │       alert_message=
    │           "Risk/Reward: " + str.tostring(riskReward, "#.##")
    │   )
    │   ```
    │
    └─ Multi-language support
        ```pine
        // Based on user preference
        alertMessage = language == "zh" ?
                       "Buy executed" :
                       "Buy executed"
        strategy.entry(..., alert_message=alertMessage)
        ```
```

## 🔄 Mixed alert approach

```
┌─ You chose: mixed alerts
│
├─ 📊 Indicator + Strategy combo
│   └─ ```pine
│       //@version=6
│       indicator("Mixed alert example", overlay=true)
│
│       // 1. alertcondition - simple signal
│       buySignal = ta.crossover(ta.sma(close, 10), ta.sma(close, 20))
│       alertcondition(buySignal, "MA crossover signal")
│
│       // 2. alert() - detailed info
│       if buySignal and barstate.isconfirmed
│           alert(
│               "MA golden cross confirmed - Price: " + str.tostring(close) +
│               " Volume: " + str.tostring(volume),
│               alert.freq_once_per_bar_close
│           )
│       ```
│
├─ 🎯 Tiered alert system
│   └─ ```pine
│       // Three-level alert system
│       level1Signal = rsi > 70  // Level 1 warning
│       level2Signal = rsi > 80 and volumeSpike  // Level 2 warning
│       level3Signal = rsi > 90 and divergence  // Level 3 warning
│
│       // Different handling by level
│       if level1Signal
│           alertcondition(true, "RSI overbought warning")
│
│       if level2Signal
│           alert("Strong overbought signal", alert.freq_once_per_bar)
│
│       if level3Signal
│           alert("Extreme overbought! Consider reversal", alert.freq_all)
│       ```
│
└─ 📈 Multi-timeframe alerts
    └─ ```pine
        // Multi-timeframe confirmation
        m5Signal = request.security(syminfo.tickerid, "5", ta.rsi(close, 14))
        h1Signal = request.security(syminfo.tickerid, "60", ta.rsi(close, 14))

        // Short-term signal
        if ta.crossover(m5Signal, 30)
            alertcondition(true, "M5 RSI crosses above 30")

        // Multi-timeframe confirmation
        if m5Signal > 30 and h1Signal > 30
            alert("Multi-timeframe bullish confirmation", alert.freq_once_per_bar_close)
        ```
```

## 📋 Alert testing checklist

### Functional tests
- [ ] Do alerts trigger under expected conditions?
- [ ] Is message content correct?
- [ ] Is the frequency setting appropriate?
- [ ] Are false positives avoided?

### Performance tests
- [ ] Are alert responses timely?
- [ ] Do excessive alerts impact performance?
- [ ] How does it perform in historical tests?

### Usability tests
- [ ] Is the alert information sufficient?
- [ ] Is it actionable?
- [ ] Is it easy for users to understand?

## ⚠️ Common issues

1. **Alerts not triggering**
   - Ensure you're on realtime bars
   - Confirm the alert is enabled
   - Verify the condition logic

2. **Duplicate alerts**
   - Adjust frequency settings
   - Add a cooldown
   - Use barstate.isconfirmed

3. **Message formatting issues**
   - Check variable names
   - Validate string concatenation
   - Control message length

4. **Performance issues**
   - Reduce alert check frequency
   - Optimize condition logic
   - Avoid heavy computations

## 💡 Summary

| Alert type | Applicable scenarios | Pros | Cons |
|---------|---------|------|------|
| alertcondition() | Simple conditions, user selects in UI | Simple, user-controlled | Static messages, limited features |
| alert() | Dynamic messages, complex conditions | Flexible, dynamic messages | Requires coding |
| Order alerts | Strategy trading | Auto-triggered, detailed info | Strategy-only |

Key to choosing an alert approach: **match requirement complexity, consider user experience, and ensure reliability**.
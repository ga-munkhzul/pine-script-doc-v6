# Decision Tree: Avoid Repainting

## ⚠️ Starting question: Does my script need to avoid repainting?

```
┌─────────────────────────────────────────────────────┐
│   🎯 Repainting definition: results differ between  │
│   historical bars and realtime bars                 │
└─────────────────────────────────────────────────────┘
    │
    └─ 🤔 What's your use case?
        │
        ├─ Live trading/alerts
        │   └─ ✅ Must avoid repainting
        │       ↓
        │
        └─ Historical analysis/backtesting
            └─ ➖ Some repainting is acceptable
```

## 🚫 Path when repainting must be avoided

```
┌─ You chose: Must avoid repainting
│
├─ 📊 When to confirm signals?
│   │
│   ├─ Can wait until bar close
│   │   └─ ✅ Approach 1: Use confirmed bars
│   │       ```pine
│   │       // Use previous bar's values (confirmed)
│   │       confirmedClose = close[1]
│   │       confirmedHigh = high[1]
│   │       ```
│   │
│   └─ Need current bar signal
│       └─ ✅ Approach 2: Use barstate.isconfirmed
│           ```pine
│           // Wait for the current bar to close
│           buySignal = ta.crossover(close, ma) and barstate.isconfirmed
│           ```
│
├─ 🔔 Alerts?
│   │
│   ├─ Simple condition alert
│   │   └─ ✅ Use alertcondition + barstate.isconfirmed
│   │       ```pine
│   │       condition = ta.crossover(close[1], ma[1])
│   │       alertcondition(condition, "Golden cross signal")
│   │       ```
│   │
│   └─ Complex message alert
│       └─ ✅ Use alert() + frequency control
│           ```pine
│           if buySignal and barstate.isconfirmed
│               alert("Buy confirmed: " + str.tostring(close),
│                      alert.freq_once_per_bar_close)
│           ```
│
├─ 📈 Indicator calculations?
│   │
│   ├─ Using realtime data
│   │   └─ ⚠️ Risk: will repaint
│   │       ```pine
│   │       // This will repaint!
│   │       ma = ta.sma(close, 20)  // close changes on realtime bars
│   │       ```
│   │
│   └─ Use confirmed data
│       └─ ✅ Safe approach
│           ```pine
│           // Will not repaint
│           ma = ta.sma(close, 20)
│           confirmedMA = ma[1]  // Use confirmed MA value
│           ```
│
└─ 🔄 Multi-timeframe (MTF) data?
    │
    ├─ Need other timeframe data
    │   └─ ✅ Safe use of request.security()
    │       ```pine
    │       // Correct: use offset to avoid future leak
    │       higherTF = request.security(syminfo.tickerid, "1D",
    │                                  close[1], lookahead=barmerge.lookahead_on)
    │       ```
    │
    └─ Using defaults
        └─ ⚠️ Danger: may repaint
            ```pine
            // Wrong: causes future leak
            higherTF = request.security(syminfo.tickerid, "1D", close)
            ```
```

## 📊 Path when some repainting is acceptable

```
┌─ You chose: Some repainting is acceptable
│
├─ 📈 Which repainting type?
│   │
│   ├─ Realtime indicator fluctuations (e.g., MACD, RSI)
│   │   └─ ✅ Acceptable: expected behavior
│   │       💡 Note: most indicators fluctuate on realtime bars
│   │
│   ├─ Volume analysis
│   │   └─ ✅ Acceptable: requires realtime data
│   │
│   ├─ Price level display
│   │   └─ ✅ Acceptable: realtime update is valuable
│   │
│   └─ Historical signal repositioning
│       └─ ❌ Not acceptable: misleading
│           💡 Fix: use var or fixed position
│           ```pine
│           // Persist signal price to avoid moving
│           var float signalPrice = na
│           if buySignal and na(signalPrice)
│               signalPrice := close
│           ```
│
├─ 🎯 Purpose?
│   │
│   ├─ Realtime monitoring
│   │   └─ ✅ Repainting is useful
│   │       ```pine
│   │       // Realtime volume analysis
│   │       plot(volume, color=volume > ta.sma(volume, 20) ? color.green : color.red)
│   │       ```
│   │
│   └─ Historical backtest
│       └─ ⚠️ Use with caution
│           💡 Use calc_on_every_tick=false
│           ```pine
│           strategy("Strategy", calc_on_every_tick=false)
│           ```
│
└─ 📝 User disclosure?
    ├─ Yes → ✅ Document repainting behavior in description
    └─ No → ⚠️ Recommend adding a note
        ```markdown
        ## Notes
        - This indicator repaints on realtime bars
        - Signals are confirmed at bar close
        ```
```

## 🔍 How to detect repainting

### 1. Visual inspection
```
On the chart:
├─ Are historical signals stable?
├─ Do histories change after switching timeframes?
└─ Do signals disappear/move in realtime?
```

### 2. Code review
```pine
// Checklist:
□ Using realtime close/high/low values?
□ Using timenow?
□ Using request.security() without offset?
□ Using varip?
□ Plotting in loops?
□ Using calc_on_every_tick=true?
```

### 3. Test method
```pine
//@version=6
indicator("Repaint Detector")
// Test at different timeframes
// Observe whether historical signals are stable
testSignal = ta.sma(close, 10) > ta.sma(close, 20)
plotshape(testSignal, style=shape.circle, location=location.bottom)
```

## 💡 Best practices summary

### ✅ Safe practices
1. **Use offset references**
   ```pine
   confirmedValue = value[1]  // simplest and effective
   ```

2. **Use barstate.isconfirmed**
   ```pine
   if condition and barstate.isconfirmed
       // do action
   ```

3. **Safe multi-timeframe requests**
   ```pine
   request.security(..., value[1], lookahead=barmerge.lookahead_on)
   ```

4. **Clearly mark repainting behavior**
   ```pine
   // annotate that this repaints
   realtimeValue = ta.sma(close, 20)  // Note: will repaint on realtime
   ```

### ❌ Dangerous practices
1. **Using timenow**
   ```pine
   // Wrong: timenow causes future leak
   plot(timenow, "Current time")
   ```

2. **request.security() without safeguards**
   ```pine
   // Wrong: may cause future leak
   highTF = request.security(syminfo.tickerid, "D", high)
   ```

3. **Plotting on realtime bars**
   ```pine
   // Wrong: signal position will move
   if condition
       label.new(bar_index, high, "Signal")
   ```

## Special cases

### 1. Strategy scripts
```pine
strategy("No-repaint Strategy", calc_on_every_tick=false,
         process_orders_on_close=true)
// Process orders only at close to avoid repainting
```

### 2. Multi-timeframe
```pine
// Safe high-timeframe data request
higherTF_data = request.security(syminfo.tickerid, "1D",
    ta.sma(close, 20)[1],  // Use offset
    lookahead=barmerge.lookahead_on  // Explicit lookahead
)
```

### 3. Realtime display vs historical analysis
```pine
// Conditional display: realtime needs it, history doesn't
displayValue = barstate.isrealtime ? realTimeValue : confirmedValue
plot(displayValue)
```

## Decision flow chart summary

```mermaid
graph TD
    A[Start: Need to avoid repainting?] --> B{Usage}
    B -->|Live trading/alerts| C[Must avoid]
    B -->|Historical analysis| D[Acceptable]

    C --> E{Confirmation timing}
    E -->|Can wait| F[Use confirmed data<br>close[1]]
    E -->|Need current| G[barstate.isconfirmed]

    C --> H[MTF data]
    H --> I[request.security + offset]

    D --> J{Repainting type}
    J -->|Normal indicator fluctuation| K[Acceptable]
    J -->|Signal repositioning| L[Avoid]

    K --> M[Add note]
    L --> N[Use var to fix position]
```

Remember: there's no absolute good or bad — only what fits. Understand the cause of repainting and choose appropriately.
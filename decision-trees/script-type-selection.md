# Script Type Selection Decision Tree

## 🎯 Starting question: What type of script do I need to create?

```
┌─────────────────────────────────────────────────────┐
│         🤔 What do you want to create?              │
└─────────────────────────────────────────────────────┘
    │
    ├─ 📊 Analyze and display data
    │   │
    │   ├─ Only display charts and indicators?
    │   │   └─ ✅ **Use Indicator**
    │   │       ```pine
    │   │       indicator("My Indicator", overlay=true)
    │   │       ```
    │   │
    │   └─ Need reusable functionality for other scripts?
    │       └─ ✅ **Use Library**
    │           ```pine
    │           library("My Library", 2, overlay=false)
    │           export myFunction() =>
    │               // implementation
    │           ```
    │
    └─ 💰 Simulated trading and backtesting
        │
        └─ ✅ **Use Strategy**
            ```pine
            strategy("My Strategy", overlay=true)
            ```
```

## Detailed decision flow

### 📊 Indicator path

```
┌─ You chose Indicator
│
├─ 🎨 Display on main chart or separate pane?
│   ├─ Main chart (on price chart)
│   │   └─ indicator("Name", overlay=true)
│   └─ Separate pane (independent window)
│       └─ indicator("Name", overlay=false)
│
├─ 📈 Need multiple outputs?
│   ├─ Yes → consider plot count limit (max 55)
│   └─ No → a single plot is enough
│
├─ 🔔 Need alerts?
│   ├─ Simple alerts → use alertcondition()
│   │   ```pine
│   │   alertcondition(buySignal, "Buy signal")
│   │   ```
│   └─ Complex alerts → use alert()
│       ```pine
│       if buySignal
│           alert("Buy signal triggered!", alert.freq_once_per_bar)
│       ```
│
└─ 💡 Special requirements?
    ├─ Need input parameters → use input.*()
    ├─ Need color customization → use input.color()
    └─ Need style options → use input.style()
```

### 💰 Strategy path

```
┌─ You chose Strategy
│
├─ 💵 Trade direction?
│   ├─ Long only
│   │   └─ strategy("Name", default_qty_type=strategy.percent_of_equity, default_qty_value=100)
│   ├─ Short only
│   │   └─ strategy("Name", shorttitle="Short", default_qty_type=strategy.percent_of_equity)
│   └─ Both directions
│       └─ strategy("Name", default_qty_type=strategy.percent_of_equity)
│
├─ ⏰ Order execution timing?
│   ├─ Close only
│   │   └─ calc_on_every_tick=false
│   └─ Every tick
│       └─ calc_on_every_tick=true
│       ⚠️ May cause repainting
│
├─ 💰 Position sizing?
│   ├─ Fixed quantity
│   │   └─ default_qty_type=strategy.fixed
│   ├─ Percent of equity
│   │   └─ default_qty_type=strategy.percent_of_equity
│   └─ Fixed cash
│       └─ default_qty_type=strategy.cash
│
├─ 📊 Fees and slippage?
│   ├─ Use default
│   │   └─ commission_type=strategy.commission.percent
│   └─ Custom
│       └─ Set commission_value, slippage
│
└─ 🎯 Exit strategy?
    ├─ Fixed TP/SL
    │   └─ strategy.exit("Exit", "Entry", profit=100, loss=50)
    ├─ Dynamic exit
    │   └─ strategy.close("Entry", when=exitCondition)
    └─ Time-based exit
        └─ strategy.close_all(when=time >= timestamp(syminfo.tickerid, "2300-01-01"))
```

### 📚 Library path

```
┌─ You chose Library
│
├─ 📦 What do you need to export?
│   ├─ Functions
│   │   ```pine
│   │   export calculateRSI(source, length) =>
│   │       ta.rsi(source, length)
│   │   ```
│   ├─ Types
│   │   ```pine
│   │   export type MyType
│   │       float value
│   │       string name
│   │   ```
│   └─ Enums
│       ```pine
│       export enum Mode
│           FAST = 1
│           SLOW = 2
│       ```
│
├─ 🔄 Versioning?
│   └─ library("Name", version)
│       ```pine
│       library("My Toolkit", 2)
│       ```
│
├─ 📥 Import and use?
│   ```pine
│   import myLib as lib from "MyLibrary/1"
│   result = lib.calculateRSI(close, 14)
│   ```
│
└─ 💡 Best practices
    ├─ Group related functionality together
    ├─ Clear function documentation
    ├─ Sensible versioning
    └─ Avoid circular dependencies
```

## Decision checklist

### Indicator use cases
- [ ] Need to display technical indicators
- [ ] Need to analyze price action
- [ ] Need data visualization
- [ ] No actual trading involved

### Strategy use cases
- [ ] Need simulated trading
- [ ] Need backtesting
- [ ] Need performance metrics
- [ ] Need order management

### Library use cases
- [ ] Code needs reuse
- [ ] Create general-purpose utilities
- [ ] Organize complex projects
- [ ] Provide API surface

## Conversion possibilities

```
┌──────────────────────┐    ┌──────────────────────┐
│      Indicator       │    │       Strategy       │
│   (can become a      │◄──►│   (can be simplified │
│       Strategy)      │    │     to Indicator)    │
└──────────────────────┘    └──────────────────────┘
                                      │
                                      ▼
                               ┌──────────────────────┐
                               │       Library        │
                               │ (extract shared code)│
                               └──────────────────────┘
```

## Common mistakes to avoid

1. ❌ Using strategy.* functions inside an Indicator
2. ❌ Missing risk controls in a Strategy
3. ❌ Accessing chart data directly inside a Library
4. ❌ Misusing the overlay parameter settings

## Example code templates

### Indicator template
```pine
//@version=6
indicator("My Indicator", shorttitle="MI", overlay=true, timeframe="", timeframe_gaps=false)

// Inputs
len = input.int(14, "Length")
src = input.source(close, "Source")

// Calculations
ma = ta.sma(src, len)

// Plotting
plot(ma, color=color.blue, title="MA")
```

### Strategy template
```pine
//@version=6
strategy("My Strategy", overlay=true,
         default_qty_type=strategy.percent_of_equity,
         default_qty_value=100,
         commission_type=strategy.commission.percent,
         commission_value=0.1)

// Inputs
len = input.int(14, "MA length")
risk = input.float(2.0, "Risk %") / 100

// Calculations
ma = ta.sma(close, len)
longCond = ta.crossover(close, ma)
shortCond = ta.crossunder(close, ma)

// Trading logic
if longCond
    strategy.entry("Long", strategy.long)
if shortCond
    strategy.entry("Short", strategy.short)

// Risk management
stopLoss = strategy.position_avg_price * (1 - risk)
takeProfit = strategy.position_avg_price * (1 + risk * 2)
strategy.exit("Exit", "Long", stop=stopLoss, limit=takeProfit)
```

### Library template
```pine
//@version=6
library("Technical Indicators Library", 2)

// Exported types and functions
export type MACD
    float macd
    float signal
    float histogram

export calculateMACD(source, fastLength, slowLength, signalLength) =>
    fastMA = ta.ema(source, fastLength)
    slowMA = ta.ema(source, slowLength)
    macdLine = fastMA - slowMA
    signalLine = ta.sma(macdLine, signalLength)
    histogram = macdLine - signalLine
    [macdLine, signalLine, histogram]
```
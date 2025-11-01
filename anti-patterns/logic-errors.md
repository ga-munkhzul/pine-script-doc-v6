# Anti-patterns: Logic Errors

Logic errors are the most subtle issues in Pine Script. The code runs, but the results don’t match expectations. Here are common logic errors.

## 1. Incorrect condition ordering

### ❌ Bad example: wrong order of conditions
```pine
//@version=6
indicator("Error: Condition order")

// ❌ Wrong order; the second condition is never evaluated
if close > ta.sma(close, 50)
    strategy.entry("Long", strategy.long)
else if close > ta.sma(close, 20)  // Will never execute!
    strategy.entry("Long", strategy.long)
```

### 🚨 Why it's a problem
- `close > 50MA` subsumes the case `close > 20MA`
- The second condition will never be true
- Causes missed trade opportunities

### ✅ Do this instead: order from most restrictive to least
```pine
//@version=6
indicator("Correct: Condition order")

// ✅ Start with the most restrictive condition
if close > ta.sma(close, 50) and ta.rsi(close, 14) < 30
    strategy.entry("Strong Buy", strategy.long)
else if close > ta.sma(close, 20) and ta.rsi(close, 14) < 50
    strategy.entry("Buy", strategy.long)
else if close > ta.sma(close, 10)
    strategy.entry("Weak Buy", strategy.long)
```

## 2. Confusing state management

### ❌ Bad example: inconsistent state
```pine
//@version=6
indicator("Error: State confusion")

// ❌ Using multiple variables to track the same state
inLongPosition = strategy.position_size > 0
longEntryBar = 0
inTrade = false

if buySignal
    strategy.entry("Long", strategy.long)
    inLongPosition := true
    inTrade := true
    longEntryBar := bar_index

// States may desync, causing logic bugs
if inLongPosition and not inTrade
    // Contradictory state
```

### 🚨 Why it's a problem
- Multiple variables tracking the same state easily fall out of sync
- Some branches update only a subset of variables
- Leads to inconsistent state and logic errors

### ✅ Do this instead: single source of truth
```pine
//@version=6
indicator("Correct: Single state")

// ✅ Use one variable as the source of truth
positionSize = strategy.position_size
inLongPosition = positionSize > 0

// Or use an enum-like pattern to define states clearly
var state = 0  // 0=flat, 1=long, 2=short

if buySignal and state == 0
    strategy.entry("Long", strategy.long)
    state := 1
else if sellSignal and state == 1
    strategy.close_all()
    state := 0
```

## 3. Timeframe confusion

### ❌ Bad example: mixing conditions from different timeframes
```pine
//@version=6
indicator("Error: Timeframe confusion")

// ❌ Mixing conditions from different timeframes
m5_cross = ta.crossover(ta.sma(close, 10), ta.sma(close, 20))
h1_trend = close > ta.sma(close, 50)  // This is 5-minute data!

// Incorrectly assuming h1_trend is the hourly trend
if m5_cross and h1_trend
    strategy.entry("Long", strategy.long)
```

### 🚨 Why it's a problem
- All calculations happen on the chart's timeframe
- `close > ta.sma(close, 50)` is a 5-minute trend, not hourly
- Lowers signal quality

### ✅ Do this instead: make timeframes explicit
```pine
//@version=6
indicator("Correct: Explicit timeframe")

// ✅ Explicitly fetch higher timeframe data
h1_close = request.security(syminfo.tickerid, "60", close[1])
h1_trend = h1_close > request.security(syminfo.tickerid, "60",
                                      ta.sma(close, 50)[1])

m5_cross = ta.crossover(ta.sma(close, 10), ta.sma(close, 20))

// Clear multi-timeframe condition
if m5_cross and h1_trend
    strategy.entry("Long", strategy.long)
```

## 4. Ignoring barstate

### ❌ Bad example: ignoring barstate
```pine
//@version=6
indicator("Error: Ignoring barstate")

// ❌ Not checking barstate
buySignal = ta.rsi(close, 14) < 30
if buySignal
    alert("Buy signal!", alert.freq_once_per_bar)
    // May trigger multiple times on a realtime bar

// Drawing future signals
if buySignal
    label.new(bar_index + 5, low, "Projection", color.green)
```

### 🚨 Why it's a problem
- Fails to distinguish realtime vs historical bars
- May trigger repeatedly on realtime bars
- Drawing into the future may be disallowed

### ✅ Do this instead: use barstate
```pine
//@version=6
indicator("Correct: Use barstate")

// ✅ Distinguish realtime and history
buySignal = ta.rsi(close, 14) < 30

if buySignal and barstate.isconfirmed
    alert("Buy signal!", alert.freq_once_per_bar_close)

// Draw only on historical bars or the current one
if buySignal and barstate.ishistory or barstate.islast
    label.new(bar_index, low, "Signal", color.green)
```

## 5. Variable scope errors

### ❌ Bad example: messy variable scope
```pine
//@version=6
indicator("Error: Scope")

// ❌ Declaring a variable inside a conditional block
if condition
    myValue = ta.sma(close, 20)
    plot(myValue)  // Error: myValue not visible outside the block

// ❌ Loop variable leakage
for i = 0 to 10
    loopValue = i
plot(loopValue)  // Error: not accessible after the loop
```

### 🚨 Why it's a problem
- Variables declared inside a conditional have limited scope in Pine Script
- Loop variables are not accessible after the loop ends
- Leads to compile errors or unexpected behavior

### ✅ Do this instead: declare in the correct scope
```pine
//@version=6
indicator("Correct: Scope")

// ✅ Declare the variable outside the block
var float myValue = na
if condition
    myValue := ta.sma(close, 20)
plot(myValue)

// ✅ Use a local variable inside the loop
var float[] results = array.new<float>()
for i = 0 to 10
    tempValue = ta.sma(close, i + 5)
    array.push(results, tempValue)
```

## 6. Ignoring na handling

### ❌ Bad example: not handling na values
```pine
//@version=6
indicator("Error: Ignoring na")

// ❌ Not checking for na
sma20 = ta.sma(close, 20)
rsi14 = ta.rsi(close, 14)

// Early values can be na, causing issues
signal = sma20 > rsi14  // If either is na, result is na

// Using na values in calculations
avg = (sma20 + rsi14) / 2  // na propagates
```

### 🚨 Why it's a problem
- Uninitialized/insufficient data in Pine Script is na
- Arithmetic with na yields na
- Not handling na leads to odd initial indicator output

### ✅ Do this instead: handle na values
```pine
//@version=6
indicator("Correct: Handle na")

sma20 = ta.sma(close, 20)
rsi14 = ta.rsi(close, 14)

// ✅ Use nz() to provide defaults
signal = nz(sma20, 0) > nz(rsi14, 50)

// ✅ Or check explicitly
if not na(sma20) and not na(rsi14)
    signal := sma20 > rsi14
else
    signal := false

// ✅ Skip na values during aggregation
validCount = 0
total = 0.0
if not na(sma20)
    total += sma20
    validCount += 1
if not na(rsi14)
    total += rsi14
    validCount += 1

avg = validCount > 0 ? total / validCount : 0.0
```

## 7. Wrong accumulation logic

### ❌ Bad example: incorrect accumulation
```pine
//@version=6
indicator("Error: Accumulation logic")

// ❌ Incorrect asymmetric accumulation
var float cumulative = 0
if close > ta.sma(close, 20)
    cumulative += close
else
    cumulative -= close  // Error: asymmetric

// ❌ Wrong averaging approach
var float sum = 0
var int count = 0
if barstate.isconfirmed
    sum += close
    count += 1
    // Forgot to bound count
```

### 🚨 Why it's a problem
- Asymmetric accumulation introduces bias
- Average calculation has no window limit
- Long runs can cause values to grow unbounded

### ✅ Do this instead: correct accumulation
```pine
//@version=6
indicator("Correct: Accumulation logic")

// ✅ Symmetric accumulation
var float cumulative = 0
if close > ta.sma(close, 20)
    cumulative += math.abs(close - ta.sma(close, 20))
else
    cumulative -= math.abs(close - ta.sma(close, 20))

// ✅ Window-limited average
var float[] values = array.new<float>(20, 0.0)
if barstate.isconfirmed
    array.unshift(values, close)
    if array.size(values) > 20
        array.pop(values)

avg = array.avg(values)
```

## 8. Ignoring type conversions

### ❌ Bad example: implicit type conversions
```pine
//@version=6
indicator("Error: Type conversion")

// ❌ Mixing int and float may produce surprises
intPart = 100
floatPart = 0.5
result = intPart + floatPart  // OK

// ❌ Wrong division
result = 5 / 10  // Result is 0 (integer division)!

// ❌ Mixing strings and numbers
message = "Price: " + close  // close needs explicit conversion
```

### 🚨 Why it's a problem
- Pine Script has particular division rules
- String concatenation needs explicit conversion
- Type mismatches can produce unexpected results

### ✅ Do this instead: convert explicitly
```pine
//@version=6
indicator("Correct: Type conversion")

// ✅ Convert explicitly to ensure proper types
intPart = 100
floatPart = 0.5
result = float(intPart) + floatPart

// ✅ Use floating-point division
result = 5.0 / 10.0  // Result is 0.5

// ✅ String concatenation
message = "Price: " + str.tostring(close, "#.##")
```

## 9. Wrong loop logic

### ❌ Bad example: loop boundary errors
```pine
//@version=6
indicator("Error: Loop boundaries")

// ❌ Wrong loop condition
for i = 0 to array.size(arr)  // Includes size; will go out of bounds
    value = array.get(arr, i)

// ❌ Wrong loop direction
// Intended to iterate from back to front but used the wrong form
for i = array.size(arr) to 0  // Will not execute
```

### 🚨 Why it's a problem
- Pine Script's `to` is inclusive
- Max array index is `size - 1`
- Decrementing loops must use `downto`

### ✅ Do this instead: correct loops
```pine
//@version=6
indicator("Correct: Loop boundaries")

// ✅ Correct loop bounds
for i = 0 to array.size(arr) - 1
    value = array.get(arr, i)

// ✅ Correct decrementing loop
for i = array.size(arr) - 1 downto 0
    value = array.get(arr, i)
```

## 10. Ignoring operator precedence

### ❌ Bad example: wrong operator precedence
```pine
//@version=6
indicator("Error: Operator precedence")

// ❌ Misunderstanding operator precedence
value = close > high or low < ta.sma(close, 20)
// Actually executed as: (close > high) or (low < ta.sma(close, 20))

// ❌ Confusing condition
signal = rsi > 50 and close > ma or volume > avgVolume
// Actually executed as: (rsi > 50 and close > ma) or volume > avgVolume
```

### 🚨 Why it's a problem
- `>` has higher precedence than `or`
- `and` has higher precedence than `or`
- Parentheses are needed to make intent explicit

### ✅ Do this instead: use parentheses to make order explicit
```pine
//@version=6
indicator("Correct: Operator precedence")

// ✅ Clear evaluation order
value = (close > high) or (low < ta.sma(close, 20))

// ✅ Use parentheses for complex conditions
signal = (rsi > 50 and close > ma) or volume > avgVolume
// Or even clearer
signal = (rsi > 50 and close > ma) or (volume > avgVolume)
```

## Logic error checklist

1. Is the condition order correct? (from most restrictive to least)
2. Is state consistent? (single source of truth)
3. Are timeframes explicit?
4. Are you handling barstate?
5. Are variable scopes correct?
6. Are na values handled?
7. Is accumulation logic correct?
8. Are type conversions explicit?
9. Are loop bounds correct?
10. Is operator precedence explicit?

## Debugging tips

1. Use plot to inspect values
   ```pine
   plotdebug(someValue, "Debug value", display=display.data_window)
   ```

2. Add labels to confirm conditions
   ```pine
   if someCondition
       label.new(bar_index, high, "Condition met", color.red)
   ```

3. Check for na
   ```pine
   if na(someValue)
       runtime.error("na encountered!")
   ```

4. Count for validation
   ```pine
   var int count = 0
   if condition
       count += 1
   plotchar(count, "Count", "", location.top)
   ```

Remember: Logic errors are the hardest to find—test, verify, and think!
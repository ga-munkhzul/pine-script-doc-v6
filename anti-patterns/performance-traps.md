# Anti-patterns: Performance Traps

Performance issues can make scripts slow, hit computation limits, or even fail to run correctly. Here are common performance pitfalls.

## 1. Repeated computation inside loops

### ❌ Bad example: recomputing inside the loop
```pine
//@version=6
indicator("Error: Recompute inside loop")

// ❌ Recomputing the same value on each iteration
var float[] results = array.new<float>()
for i = 0 to 50
    // Recalculate MA each time — very inefficient
    ma20 = ta.sma(close, 20)
    value = close - ma20
    array.push(results, value)
```

### 🚨 Why it's a problem
- `ta.sma(close, 20)` is computed 51 times with the same inputs
- As loop count increases, compute time grows rapidly
- May trigger Pine Script’s computation time limits

### ✅ Do this instead: precompute values
```pine
//@version=6
indicator("Correct: Precompute values")

// ✅ Compute once outside the loop
ma20 = ta.sma(close, 20)
var float[] results = array.new<float>()

for i = 0 to 50
    // Use the precomputed value
    value = close - ma20
    array.push(results, value)
```

## 2. Unbounded array growth

### ❌ Bad example: array grows without bound
```pine
//@version=6
indicator("Error: Memory leak")

// ❌ Array keeps growing, never cleaned up
var float[] priceHistory = array.new<float>()

// Add on every confirmed bar, never remove
if barstate.isconfirmed
    array.push(priceHistory, close)

// Can exhaust memory
avgPrice = array.avg(priceHistory)
plot(avgPrice, "Average price")
```

### 🚨 Why it's a problem
- Array size grows until Pine Script limits are hit
- Each average requires traversing the entire array
- Performance degrades over time

### ✅ Do this instead: cap the array size
```pine
//@version=6
indicator("Correct: Fixed-size array")

// ✅ Cap the maximum size
var float[] priceHistory = array.new<float>()
maxSize = 200

if barstate.isconfirmed
    array.unshift(priceHistory, close)
    // Keep array size bounded
    if array.size(priceHistory) > maxSize
        array.pop(priceHistory)

// Efficient average
avgPrice = array.avg(priceHistory)
plot(avgPrice, "Average price")
```

## 3. Overusing request.security()

### ❌ Bad example: frequent cross-timeframe requests
```pine
//@version=6
indicator("Error: Frequent MTF requests")

// ❌ Request multiple timeframes on every tick
m5_rsi = request.security(syminfo.tickerid, "5", ta.rsi(close, 14))
m15_rsi = request.security(syminfo.tickerid, "15", ta.rsi(close, 14))
h1_rsi = request.security(syminfo.tickerid, "60", ta.rsi(close, 14))
h4_rsi = request.security(syminfo.tickerid, "240", ta.rsi(close, 14))

// Used frequently in strategy logic
if m5_rsi > 50 and m15_rsi > 50 and h1_rsi > 50 and h4_rsi > 50
    // trading logic
```

### 🚨 Why it's a problem
- `request.security()` is expensive
- Calling it on every tick severely impacts performance
- Multiple calls can hit server-side limits

### ✅ Do this instead: cache and update conditionally
```pine
//@version=6
indicator("Correct: Cache MTF data")

// ✅ Use var to cache values
var float m5_rsi = na
var float m15_rsi = na
var float h1_rsi = na
var float h4_rsi = na

// Update only when needed
updateMTF = barstate.isconfirmed

if updateMTF
    // Batch the requests
    m5_rsi := request.security(syminfo.tickerid, "5", ta.rsi(close, 14)[1])
    m15_rsi := request.security(syminfo.tickerid, "15", ta.rsi(close, 14)[1])
    h1_rsi := request.security(syminfo.tickerid, "60", ta.rsi(close, 14)[1])
    h4_rsi := request.security(syminfo.tickerid, "240", ta.rsi(close, 14)[1])

// Use cached data
if m5_rsi > 50 and m15_rsi > 50 and h1_rsi > 50 and h4_rsi > 50
    // trading logic
```

## 4. Overly deep nested loops

### ❌ Bad example: nested loops
```pine
//@version=6
indicator("Error: Nested loops")

// ❌ Nested loops — performance disaster
var float[][] matrix = matrix.new<float>(50, 50)

for i = 0 to 49
    for j = 0 to 49
        // 2500 iterations!
        matrix.set(matrix, i, j, close * (i + j))
```

### 🚨 Why it's a problem
- Nested loops are O(n²)
- 50×50 = 2500 operations
- Pine Script loop limit is ~100; this pattern will cause errors

### ✅ Do this instead: vectorize
```pine
//@version=6
indicator("Correct: Vectorized approach")

// ✅ Avoid nested loops
var float[] vector = array.new<float>()

// Use built-in functions and single loops
for i = 0 to 49
    value = close * i
    array.push(vector, value)

// If you truly need a matrix, split the work into stages
// Or reconsider whether you need that much computation
```

## 5. Unnecessary drawing objects

### ❌ Bad example: creating too many objects
```pine
//@version=6
indicator("Error: Too many drawing objects")

// ❌ Create a label for every data point
var label[] labels = array.new<label>()

// Unbounded label creation
for i = 0 to 100
    if close > ta.sma(close, 20)
        newLabel = label.new(bar_index - i, high[i], "High",
                           color.red, size.small)
        array.push(labels, newLabel)
```

### 🚨 Why it's a problem
- Label count grows without control
- Each label consumes memory
- The chart becomes very slow

### ✅ Do this instead: limit and clean up
```pine
//@version=6
indicator("Correct: Limit drawing objects")

// ✅ Cap the maximum number of labels
var label[] labels = array.new<label>()
maxLabels = 10

// Create only when needed
if close > ta.sma(close, 20)
    newLabel = label.new(bar_index, high, "High",
                       color.red, size.small)
    array.unshift(labels, newLabel)

    // Remove old labels
    if array.size(labels) > maxLabels
        oldLabel = array.pop(labels)
        label.delete(oldLabel)
```

## 6. Repeating expensive computations

### ❌ Bad example: recomputing complex indicators
```pine
//@version=6
indicator("Error: Duplicate computations")

// ❌ Compute the same indicator multiple times
if ta.rsi(close, 14) > 70
    // Overly strict nested condition
    if ta.rsi(close, 14) > 80
        // Super condition
        plotshape(1, "Overbought", style=shape.triangledown)

    // MACD also recomputed in multiple places
    [macdLine, signalLine] = ta.macd(close, 12, 26, 9)
    if macdLine > signalLine
        plotshape(1, "Divergence", style=shape.circle)
```

### 🚨 Why it's a problem
- `ta.rsi(close, 14)` is computed multiple times
- `ta.macd()` is called in several places
- Complex indicators are expensive to compute

### ✅ Do this instead: cache results
```pine
//@version=6
indicator("Correct: Cache computations")

// ✅ Compute once, reuse many times
rsi14 = ta.rsi(close, 14)
[macdLine, signalLine] = ta.macd(close, 12, 26, 9)

// Use cached values
if rsi14 > 70
    if rsi14 > 80
        plotshape(1, "Overbought", style=shape.triangledown)

    if macdLine > signalLine
        plotshape(1, "Divergence", style=shape.circle)
```

## 7. Unnecessary string operations

### ❌ Bad example: frequent string concatenation
```pine
//@version=6
indicator("Error: Frequent string operations")

// ❌ Concatenate strings on every iteration
for i = 0 to 20
    message = "Price: " + str.tostring(close[i]) +
              " Time: " + str.format("{0,date,yyyy-MM-dd}", time[i]) +
              " RSI: " + str.tostring(ta.rsi(close[i], 14))
    // use message
```

### 🚨 Why it's a problem
- String operations are expensive in Pine Script
- Concatenating strings inside loops severely impacts performance
- Each concatenation creates a new string object

### ✅ Do this instead: minimize string work
```pine
//@version=6
indicator("Correct: Minimize string operations")

// ✅ Only create strings when needed
// Precompute numbers
rsi14 = ta.rsi(close, 14)

// Only show detailed info on the last bar
if barstate.islast
    message = "Price: " + str.tostring(close) +
              " RSI: " + str.tostring(rsi14)
    label.new(bar_index, high, message)
```

## 8. Ignoring early exit

### ❌ Bad example: unnecessary full-loop iteration
```pine
//@version=6
indicator("Error: Unnecessary full loop")

// ❌ Keep looping even after target is found
var float[] data = array.from(1, 5, 10, 15, 20, 25, 30)
target = 15
foundIndex = -1

for i = 0 to array.size(data) - 1
    if array.get(data, i) == target
        foundIndex := i
    // Continues looping, wasting time
```

### 🚨 Why it's a problem
- Continues looping after the target is found
- No break statement used (Pine Script does not support it, but can be simulated)
- Wastes unnecessary compute resources

### ✅ Do this instead: conditional exit
```pine
//@version=6
indicator("Correct: Conditional exit")

// ✅ Exit once the target is found
var float[] data = array.from(1, 5, 10, 15, 20, 25, 30)
target = 15
foundIndex = -1

for i = 0 to array.size(data) - 1
    if array.get(data, i) == target
        foundIndex := i
        break  // Use break to exit early
    // Not executed after found
```

## 9. Overusing historical references

### ❌ Bad example: deep historical references
```pine
//@version=6
indicator("Error: Deep history references")

// ❌ Referencing too much history
volatility = ta.stdev(ta.change(close, 1), 100) * 100
veryOldMA = ta.sma(close, 500)  // reference 500 bars ago
ancientHigh = ta.highest(high, 1000)  // reference 1000 bars ago

// Each reference requires accessing history
plot(volatility, "Volatility")
plot(veryOldMA, "500MA")
```

### 🚨 Why it's a problem
- Deep history requires loading lots of data
- Each calculation traverses more history
- Especially slow on lower timeframes

### ✅ Do this instead: use reasonable history lengths
```pine
//@version=6
indicator("Correct: Reasonable history length")

// ✅ Use reasonable windows
volatility = ta.stdev(ta.change(close, 1), 20) * 100  // 20 periods is sufficient
recentMA = ta.sma(close, 100)  // 100 periods is more practical
recentHigh = ta.highest(high, 50)  // Recent 50-bar high as a reference

plot(volatility, "Volatility")
plot(recentMA, "100MA")
```

## 10. Ignoring conditional computation

### ❌ Bad example: always compute everything
```pine
//@version=6
indicator("Error: Always compute")

// ❌ Compute all indicators even when not needed
rsi = ta.rsi(close, 14)
macd = ta.macd(close, 12, 26, 9)
bb = ta.bb(close, 20, 2)
stoch = ta.stoch(close, high, low, 14)

// Compute everything even if only one is displayed
showRSI = input.bool(false)
if showRSI
    plot(rsi, "RSI")
```

### 🚨 Why it's a problem
- Unnecessary indicators are computed
- Wastes CPU cycles
- Increases script load time

### ✅ Do this instead: compute conditionally
```pine
//@version=6
indicator("Correct: Conditional computation")

// ✅ Compute only what is needed
showRSI = input.bool(false)
showMACD = input.bool(false)
showBB = input.bool(false)

// Conditional computation
var float rsi = na
var float macd = na
var [bbUpper, bbMiddle, bbLower] = [na, na, na]

if showRSI
    rsi := ta.rsi(close, 14)
    plot(rsi, "RSI")

if showMACD
    [macd, signal, hist] = ta.macd(close, 12, 26, 9)
    plot(macd, "MACD")

if showBB
    [bbUpper, bbMiddle, bbLower] = ta.bb(close, 20, 2)
    plot(bbUpper, "BB Upper")
    plot(bbMiddle, "BB Middle")
    plot(bbLower, "BB Lower")
```

## Performance optimization checklist

1. Is there repeated computation inside loops?
2. Do arrays risk unbounded growth?
3. Are request.security() calls too frequent?
4. Any unnecessary nested loops?
5. Is the number of drawing objects controlled?
6. Are complex indicators cached?
7. Are string operations minimized?
8. Are historical references reasonable?
9. Is conditional computation used where appropriate?
10. Can loops exit early once the target is found?

## Golden rules of performance

1. Precompute: compute outside loops, use inside
2. Limit sizes: cap arrays and object counts
3. Cache data: memoize expensive results
4. Conditional execution: don’t compute what you don’t need
5. Batch operations: prefer built-ins over manual loops
6. Clean up: regularly delete unused objects
7. Reasonable references: avoid excessive historical lookbacks

Remember: Performance optimization is a balancing act — make it correct first, then make it fast.
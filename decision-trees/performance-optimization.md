# Performance Optimization Decision Tree

## ⚡ Starting question: Is the script slow or hitting computation limits?

```
┌─────────────────────────────────────────────────────┐
│   ⚠️ Pine Script performance limits:               │
│   • Loops: at most 100 iterations                   │
│   • request.security() call frequency limits        │
│   • Each script has a compute time limit            │
│   • Drawing object limits (labels, lines, etc.)     │
└─────────────────────────────────────────────────────┘
    │
    └─ 🔍 Identify the symptom?
        │
        ├─ Loop errors/limit errors
        │   └─ ➡️ **Go to Loop optimization**
        │
        ├─ Script loads slowly
        │   └─ ➡️ **Go to Compute optimization**
        │
        ├─ Chart renders slowly
        │   └─ ➡️ **Go to Plotting optimization**
        │
        └─ Multi-timeframe data is slow
            └─ ➡️ **Go to Cross-timeframe optimization**
```

## 🔄 Loop optimization path

```
┌─ Question: Loop performance bottleneck
│
├─ 📊 Loop type?
│   │
│   ├─ for loop
│   │   └─ 🔧 Optimization strategies:
│   │       ```pine
│   │       // ❌ Wrong: unnecessary long loop
│   │       for i = 0 to 99
│   │           array.push(arr, ta.sma(close, i + 1))
│   │
│   │       // ✅ Correct: use built-in functions
│   │       values = array.new<float>()
│   │       array.push(values, ta.sma(close, 100))
│   │       ```
│   │
│   ├─ while loop
│   │   └─ 🔧 Optimization strategies:
│   │       ```pine
│   │       // ❌ Dangerous: possible infinite loop
│   │       while condition
│   │           // Handle
│   │
│   │       // ✅ Safe: add a counter
│   │       counter = 0
│   │       while condition and counter < 100
│   │           counter += 1
│   │       ```
│   │
│   └─ array loop operations
│       └─ 🔧 Use built-in methods:
│           ```pine
           // ❌ Slow: manual iteration
           sum = 0.0
           for i = 0 to array.size(arr) - 1
               sum += array.get(arr, i)

           // ✅ Fast: use built-in functions
           sum = array.sum(arr)
           avg = array.avg(arr)
│           ```
│
├─ 💡 Loop optimization tips
│   │
│   ├─ Early exit
│   │   ```pine
│   │   // Exit immediately once target found
│   │   for i = 0 to array.size(arr) - 1
│   │       if array.get(arr, i) == target
│   │           break
│   │   ```
│   │
│   ├─ Reduce in-loop computation
│   │   ```pine
│   │   // ❌ Repeated calculation
│   │   for i = 0 to len
│   │       result = ta.sma(close, 20) * i
│   │
│   │   // ✅ Precompute
│   │   baseValue = ta.sma(close, 20)
│   │   for i = 0 to len
│   │       result = baseValue * i
│   │   ```
│   │
│   ├─ Batch operations
│   │   ```pine
│   │   // Use batch methods like array.fill, array.slice
│   │   ```
│   │
│   └─ Cache results
│       ```pine
│       // Cache results using var
│       var cachedValue = na
│       if na(cachedValue)
│           cachedValue := expensiveCalculation()
│       ```
│
└─ 📈 Alternatives to loops
    ├─ Use built-ins instead
    ├─ Use math formulas instead
    └─ Use vectorized operations
```

## ⚙️ Compute optimization path

```
┌─ Question: Script computation is slow
│
├─ 🎯 Computation frequency optimization
│   │
│   ├─ Compute on every bar?
│   │   └─ 📝 Check calc_on_every_tick
│   │       ```pine
│   │       // ❌ Recalculate on every tick in realtime
│   │       strategy("Strategy", calc_on_every_tick=true)
│   │
│   │       // ✅ Compute only on bar close
│   │       strategy("Strategy", calc_on_every_tick=false)
│   │       ```
│   │
│   └─ Reduce unnecessary computation
│       ```pine
│       // ❌ Compute every time
│       ma20 = ta.sma(close, 20)
│       ma50 = ta.sma(close, 50)
│       ma200 = ta.sma(close, 200)
│       even if not needed
│
│       // ✅ Conditional computation
│       ma20 = input.bool(true, "Show MA20") ? ta.sma(close, 20) : na
│       ```
│
├─ 📊 Math operations optimization
│   │
│   ├─ Avoid complex operations
│   │   ```pine
│   │   // ❌ Slow: repeated power operations
│   │   result = math.pow(x, 2) + math.pow(y, 2)
│   │
│   │   // ✅ Fast: direct multiplication
│   │   result = x * x + y * y
│   │   ```
│   │
│   ├─ Use approximations
│   │   ```pine
│   │   // ❌ Precise but slow
│   │   sqrt = math.sqrt(value)
│   │
│   │   // ✅ Approximate but fast
│   │   sqrt = value * 0.5  // in some contexts
│   │   ```
│   │
│   └─ Avoid type conversions
│       ```pine
│       // ❌ Multiple conversions
│       result = int(string(floatValue))
│
│       // ✅ Keep types consistent
│       result = int(floatValue)
│       ```
│
├─ 🔄 Function call optimization
│   │
│   ├─ Cache function results
│   │   ```pine
│   │   // ❌ Repeated calls
│   │   plot(ta.rsi(close, 14))
│   │   plot(ta.rsi(close, 14) - 30)
│   │
│   │   // ✅ Cache result
│   │   rsi14 = ta.rsi(close, 14)
│   │   plot(rsi14)
│   │   plot(rsi14 - 30)
│   │   ```
│   │
│   ├─ Reduce custom function calls
│   │   ```pine
│   │   // ❌ Recomputes on each call
│   │   calculateMA() =>
│   │       ta.sma(close, 20)
│   │
│   │   // ✅ Inline computation
│   │   ma20 = ta.sma(close, 20)
│   │   ```
│   │
│   └─ Use efficient algorithms
│       ```pine
       // Choose algorithms with lower time complexity
│       ```
│
└─ 📝 Conditional optimization
    ├─ Short-circuit evaluation
    │   ```pine
    │   // ✅ Evaluate lightweight condition first
    │   if lightCondition and heavyCondition
    │       // Handle
    │   ```
    │
    ├─ Reduce nesting
    │   ```pine
    │   // ❌ Deep nesting
    │   if a
    │       if b
    │           if c
    │               // Handle
    │
    │   // ✅ Flatten structure
    │   if a and b and c
    │       // 处理
    │   ```
    │
    └─ Use switch instead of multiple ifs
        ```pine
        // ✅ Clearer and more efficient
        switch mode
            "MA" => ta.sma(close, len)
            "EMA" => ta.ema(close, len)
            "WMA" => ta.wma(close, len)
        => ta.sma(close, len)
        ```
```

## 🎨 Plotting optimization path

```
┌─ Question: Chart rendering is slow
│
├─ 📊 Number of drawing objects
│   │
│   ├─ Too many plot objects?
│   │   └─ 📝 Reduce or merge
│   │       ```pine
│   │       // ❌ Multiple separate plots
│   │       plot(ma20)
│   │       plot(ma50)
│   │       plot(ma200)
│   │
│   │       // ✅ Combine display
│   │       plot(ma20, "MA20", color.blue)
│   │       plot(ma50, "MA50", color.red)
│   │       // Or use the colors parameter
│   │       ```
│   │
│   ├─ Too many labels/lines?
│   │   └─ 📝 Limit count or clean up
│   │       ```pine
│   │       // ❌ Create without limit
│   │       if condition
│   │           label.new(bar_index, high, "Label")
│   │
│   │       // ✅ Limit number
│   │       var label[] labels = array.new<label>()
│   │       if condition and array.size(labels) < 10
│   │           array.push(labels, label.new(...))
│   │
│   │       // ✅ Clean old labels
│   │       if barstate.isconfirmed
│   │           label.delete(array.shift(labels))
│   │       ```
│   │
│   └─ Table updates too frequent?
│       └─ 📝 Optimize update frequency
│           ```pine
│           // ❌ Update on every tick
│           if barstate.isrealtime
│               table.cell(...)
│
│           // ✅ Update only when needed
│           if condition and barstate.isconfirmed
│               table.cell(...)
│           ```
│
├─ 🎨 Visual effects optimization
│   │
│   ├─ Complex color computation
│   │   ```pine
│   │   // ❌ Compute color every time
│   │   plotColor = rsi > 70 ? color.red :
│   │               rsi < 30 ? color.green :
│   │               color.blue
│   │
│   │   // ✅ Cache color
│   │   var plotColor = color.blue
│   │   if rsi > 70
│   │       plotColor := color.red
│   │   else if rsi < 30
│   │       plotColor := color.green
│   │   ```
│   │
│   ├─ Opacity gradient
│   │   ```pine
│   │   // ❌ Complex calculation
│   │   alpha = math.abs(rsi - 50) * 2.55
│   │
│   │   // ✅ Use step values
│   │   alpha = rsi > 60 ? 80 : rsi < 40 ? 40 : 60
│   │   ```
│   │
│   └─ Style switching
│       ```pine
       // Use style parameter to optimize display
       plot(value, style=plot.style_area)
│       ```
│
└─ 📈 Conditional plotting
    ├─ Reduce unnecessary drawing
    │   ```pine
    │   // ❌ Always draw
    │   plot(signal, "Signal")
    │
    │   // ✅ Conditional drawing
    │   plot(showSignals ? signal : na, "Signal")
    │   ```
    │
    └─ Use display parameter
        ```pine
        // Control display scope
        plot(ma, display=display.none)  // Show in data window only
        ```
```

## 📊 Cross-timeframe optimization path

```
┌─ Question: request.security() performance issues
│
├─ 🔄 Call frequency
│   │
│   ├─ Request on every bar?
│   │   └─ 📝 Reduce number of requests
│   │       ```pine
│   │       // ❌ Request every time
│   │       higherTF = request.security(..., close)
│   │
│   │       // ✅ Cache result
│   │       var higherTF = na
│   │       if barstate.isconfirmed
│   │           higherTF := request.security(..., close[1])
│   │       ```
│   │
│   └─ Request multiple values?
│       └─ 📝 Batch requests
│           ```pine
│           // ❌ Multiple requests
│           high_tf = request.security(..., high)
│           low_tf = request.security(..., low)
│           close_tf = request.security(..., close)
│
│           // ✅ Single request
│           [h, l, c] = request.security(...,
│               [high, low, close])
│           ```
│
├─ 📊 Amount of requested data
│   │
│   ├─ Requesting long timeframe data?
│   │   └─ 📝 Use lookahead=barmerge.lookahead_on
│   │       ```pine
│   │       // ✅ Explicitly set lookahead
│   │       data = request.security(...,
│           close,
│           lookahead=barmerge.lookahead_on)
│   │       ```
│   │
│   └─ Request multiple timeframes?
│       └─ 📝 Prioritize
│           ```pine
│           // Request in order of importance
│           primary = request.security(..., primaryData)
│           if time > lastRequestTime
│               secondary = request.security(..., secondaryData)
│           ```
│
└─ 💡 Caching strategies
    ├─ Use var for caching
    │   ```pine
    │   // Cache cross-timeframe data
    │   var cachedData = array.new<float>()
    │   ```
    │
    ├─ Timed updates
    │   ```pine
    │   // Update once per hour
    │   if ta.change(time('60'))
    │       cachedData := updateData()
    │   ```
    │
    └─ Update on demand
        ```pine
        // Update only when needed
        if needUpdate and barstate.isconfirmed
            updateData()
        ```
```

## 🛠️ General optimization strategies

### 1. Code structure optimization
```pine
// ❌ Scattered calculations
a = calculateA()
b = calculateB()
c = calculateC()

// ✅ Organize calculations
calculateAll() =>
    [calculateA(), calculateB(), calculateC()]
[a, b, c] = calculateAll()
```

### 2. Data structure optimization
```pine
// ❌ Multiple variables
var float val1 = na
var float val2 = na
var float val3 = na

// ✅ Use arrays
var values = array.new<float>(3, 0.0)
```

### 3. Conditional execution optimization
```pine
// ❌ Always execute
heavyCalculation()

// ✅ Execute conditionally
if needCalculation and barstate.isconfirmed
    heavyCalculation()
```

## 📊 Performance detection tools

### 1. Built-in checks
```pine
// Check computation time
start = timenow
result = heavyCalculation()
elapsed = timenow - start
if elapsed > 100  // More than 100ms
    label.new(bar_index, high, "Slow calc: " + str.tostring(elapsed) + "ms")
```

### 2. Loop counter
```pine
// Monitor loop iterations
var loopCount = 0
for i = 0 to array.size(arr) - 1
    loopCount += 1
    if loopCount > 90
        runtime.error("Approaching loop limit!")
```

### 3. Memory usage check
```pine
// Check array size
if array.size(hugeArray) > 10000
    runtime.error("Array too large!")
```

## 💡 Performance optimization checklist

- [ ] Reduce unnecessary computation
- [ ] Cache reused values
- [ ] Optimize loops (reduce iterations)
- [ ] Use built-in functions instead of custom implementations
- [ ] Limit number of drawing objects
- [ ] Optimize request.security() calls
- [ ] Use appropriate data structures
- [ ] Conditionally execute recomputations
- [ ] Simplify mathematical operations
- [ ] Avoid frequent type conversions
- [ ] Use var for persistent variables
- [ ] Batch operations instead of single operations

## ⚠️ Common performance pitfalls

1. **Overuse of request.security()**
2. **Complex calculations inside loops**
3. **Creating too many drawing objects**
4. **Updating all data on every tick**
5. **Unnecessary historical references**
6. **Deeply nested conditionals**
7. **Repeated string operations**
8. **Unnecessary data type conversions**

Remember: **Optimization is the art of balance**. While pursuing performance, also consider code readability and maintainability. Make it correct first, then make it fast.
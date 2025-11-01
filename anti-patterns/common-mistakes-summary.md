# Pine Script Common Mistakes Summary

## Error Statistics

Below are the most common mistakes in Pine Script development, ordered by frequency:

| Error type | Frequency | Impact level | Easy to detect |
|------------|-----------|--------------|----------------|
| Repainting errors | Extremely high | Severe | Hard |
| Performance issues | High | Medium | Medium |
| Logic errors | High | Severe | Hard |
| Data structure misuse | Medium | Medium | Easy |
| Type errors | Low | Severe | Easy |

## Top 10 Deadly Mistakes

### 1. **Future leak** (most severe)
```pine
// Fatal mistake
dailyClose = request.security(syminfo.tickerid, "D", close)

// Correct approach
dailyClose = request.security(syminfo.tickerid, "D", close[1])
```

### 2. **Misuse of live data**
```pine
// Causes repainting
if ta.crossover(close, ta.sma(close, 20))
    strategy.entry("Long", strategy.long)

// Wait for confirmation
if ta.crossover(close, ta.sma(close, 20)) and barstate.isconfirmed
    strategy.entry("Long", strategy.long)
```

### 3. **Infinite loops**
```pine
// Dangerous: may loop infinitely
var float value = 0
value := ta.sma(value, 10)  // Circular reference

// Correct computation
rawValue = close * 0.1
value := ta.sma(rawValue, 10)
```

### 4. **Array out-of-bounds**
```pine
// Runtime error
value = array.get(arr, array.size(arr))

// Safe access
index = array.size(arr) - 1
value = index >= 0 ? array.get(arr, index) : na
```

### 5. **Memory leaks**
```pine
// Unbounded growth
var float[] data = array.new<float>()
array.push(data, close)  // Never cleaned up

// Limit size
if array.size(data) > 100
    array.shift(data)
```

### 6. **Wrong condition order**
```pine
// Second condition never executes
if close > ma50
    // Code
else if close > ma20  // Always false
    // Code
```

### 7. **Ignoring na values**
```pine
// na propagation
result = (naValue + 10) / 2  // Result is na

// Handle na values
result = (nz(naValue, 0) + 10) / 2
```

### 8. **Performance traps**
```pine
// Repeated computation
for i = 0 to 100
    ma = ta.sma(close, 20)  // Calculated each time

// Precompute
ma = ta.sma(close, 20)
for i = 0 to 100
    // Use ma
```

### 9. **State desynchronization**
```pine
// Multiple state variables
inLong1 = strategy.position_size > 0
inLong2 = someOtherCondition  // May be inconsistent

// Single source of truth
inLong = strategy.position_size > 0
```

### 10. **Timeframe confusion**
```pine
// Mistakenly assuming this is hourly data
h1Trend = close > ta.sma(close, 50)  // Actually current timeframe

// Fetch explicitly
h1Trend = request.security(..., "60", close) > request.security(..., "60", ta.sma(close, 50))
```

## Error-Prevention Principles

### 1. **Defensive programming**
```pine
// Always check bounds
if index >= 0 and index < array.size(arr)
    value = array.get(arr, index)
```

### 2. **Explicitness principle**
```pine
// Use parentheses to make precedence explicit
condition = (rsi > 50 and close > ma) or volume > avgVolume
```

### 3. **Single responsibility**
```pine
// Each variable/function does exactly one thing
var bool inPosition = strategy.position_size > 0  // State tracking
```

### 4. **Minimalism principle**
```pine
// Compute only when needed
if showIndicator
    expensiveCalculation()
```

### 5. **Consistency principle**
```pine
// Keep naming and style consistent
ma20 = ta.sma(close, 20)
ma50 = ta.sma(close, 50)  // Consistent naming
```

## Code review checklist

### Pre-commit checks

1. Repainting checks
   - Is there an offset in request.security()?
   - Do you wait for barstate.isconfirmed?
   - Do you use timenow for historical checks?

2. Performance checks
   - Do loops perform repeated calculations?
   - Do arrays grow without bound?
   - Are request.security() calls reasonable?

3. Logic checks
   - Is the condition order correct?
   - Is state consistent?
   - Are all edge cases handled?

4. Type checks
   - Any type casts?
   - Are array types consistent?
   - Are na values handled?

5. Test checks
   - Tested on different timeframes?
   - How does it perform historically?
   - Does real-time behavior meet expectations?

## Quick fix templates

### Fix repainting
```pine
// Add an offset
value[1]

// Add confirmation
if condition and barstate.isconfirmed

// Set lookahead
request.security(..., lookahead=barmerge.lookahead_on)
```

### Fix performance
```pine
// Cache computation
var cachedValue = na
if updateCondition
    cachedValue := expensiveCalculation()

// Limit array size
if array.size(arr) > maxSize
    array.shift(arr)
```

### Fix logic
```pine
// Use parentheses
condition = (a and b) or (c and d)

// Handle na
if not na(value)
    // Use value
```

## Remember these

1. If backtests look too perfect, there's probably repainting
2. Performance issues often come from loops and arrays
3. Logic errors are hardest to spot; test more
4. Type errors are easiest; the compiler will help
5. Good code = Simple + Clear + Testable

## Advanced Recommendations

1. **Use version control**
   - Commit each major change
   - Allow rollback to a working version

2. **Write test cases**
   - Normal cases
   - Edge cases
   - Exceptional cases

3. **Add comments**
```pine
// Explain why, not just what
// Use an offset to avoid future leaks
dailyClose = request.security(..., close[1])
```

4. **Modularize code**
   - One responsibility per function
   - Extract reusable functionality

5. **Continuous learning**
   - Review others' code
   - Learn best practices
   - Keep knowledge up-to-date

---

**Final advice**: Code should not only run, but be correct, efficient, and maintainable. Think more, test more, and keep improving!
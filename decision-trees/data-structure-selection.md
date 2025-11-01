# Data Structure Selection Decision Tree

## 📦 Starting question: What type of data do I need to store?

```
┌─────────────────────────────────────────────────────┐
│   🗃️ Pine Script v6 Data Structure Selection Guide   │
│   Arrays (v4+) | Maps (v6) | Matrices (v6)         │
└─────────────────────────────────────────────────────┘
    │
    └─ 📊 Data dimension?
        │
        ├─ 1D data (list)
        │   └─ ➡️ **Array**
        │
        ├─ Key-value (dictionary)
        │   └─ ➡️ **Map**
        │
        ├─ 2D data (table/matrix)
        │   └─ ➡️ **Matrix**
        │
        └─ Simple variables
            └─ ➡️ **Basic variables**
```

## 📚 Arrays (one-dimensional lists)

```
┌─ Choose: Array - one-dimensional collection
│
├─ 🎯 Use cases?
│   │
│   ├─ Store historical data
│   │   └─ ✅ **Price/indicator series**
│   │       ```pine
│   │       // Store the closes of the most recent N bars
│   │       var float[] closePrices = array.new<float>(50)
│   │       if barstate.isconfirmed
│   │           array.unshift(closePrices, close)
│   │           if array.size(closePrices) > 50
│   │               array.pop(closePrices)
│   │       ```
│   │
│   ├─ Store signals
│   │   └─ ✅ **Buy/sell signal sequence**
│   │       ```pine
│   │       // Record all signals
│   │       var int[] signals = array.new<int>()
│   │       if buySignal
│   │           array.push(signals, 1)
│   │       if sellSignal
│   │           array.push(signals, -1)
│   │       ```
│   │
│   ├─ Store computed results
│   │   └─ ✅ **Intermediate values**
│   │       ```pine
│   │       // Store multiple computation results
│   │       var float[] results = array.new<float>()
│   │       for i = 0 to 10
│   │           array.push(results, ta.rsi(close, i + 5))
│   │       ```
│   │
│   └─ Store configuration parameters
│       └─ ✅ **Dynamic parameter list**
│           ```pine
│           // Configurable period list
│           periods = array.from(5, 10, 20, 50, 100, 200)
│           for period in periods
│               ma = ta.sma(close, period)
│               plot(ma)
│           ```
│
├─ 🔧 Array operations
│   │
│   ├─ Create and initialize
│   │   ```pine
│   │   // Empty array
│   │   arr = array.new<float>()
│   │
│   │   // Preallocate size
│   │   arr = array.new<float>(100)
│   │
│   │   // From values
│   │   arr = array.from(1, 2, 3, 4, 5)
│   │
│   │   // Fill with initial value
│   │   arr = array.new<float>(10, 0.0)
│   │   ```
│   │
│   ├─ Add elements
│   │   ```pine
│   │   // Append at end
│   │   array.push(arr, value)
│   │
│   │   // Add to front
│   │   array.unshift(arr, value)
│   │
│   │   // Insert at index
│   │   array.insert(arr, index, value)
│   │   ```
│   │
│   ├─ Access and modify
│   │   ```pine
│   │   // Get element
│   │   value = array.get(arr, index)
│   │
│   │   // Set element
│   │   array.set(arr, index, value)
│   │
│   │   // Shorthand syntax
│   │   value = arr[index]
│   │   arr[index] = value
│   │   ```
│   │
│   └─ Remove elements
│       ```pine
│       // Remove last
│       array.pop(arr)
│
│       // Remove first
│       array.shift(arr)
│
│       // Remove at index
│       array.remove(arr, index)
│
│       // Clear array
│       array.clear(arr)
│       ```
│
├─ 📊 Array utility functions
│   │
│   ├─ Search and filter
│   │   ```pine
│   │   // Find element
│   │   index = array.indexof(arr, targetValue)
│   │
│   │   // Contains check
│   │   contains = array.includes(arr, targetValue)
│   │
│   │   // Filter (Pine Script v6)
│   │   filtered = array.filter(arr, val => val > threshold)
│   │   ```
│   │
│   ├─ Statistics
│   │   ```pine
│   │   // Sum
│   │   sum = array.sum(arr)
│   │
│   │   // Average
│   │   avg = array.avg(arr)
│   │
│   │   // Max/Min
│   │   max = array.max(arr)
│   │   min = array.min(arr)
│   │
│   │   // Median
│   │   median = array.median(arr)
│   │   ```
│   │
│   ├─ Sorting
│   │   ```pine
│   │   // Ascending sort
│   │   array.sort(arr, order.ascending)
│   │
│   │   // Descending sort
│   │   array.sort(arr, order.descending)
│   │   ```
│   │
│   └─ Slicing and merging
│       ```pine
│       // Slice
│       subArray = array.slice(arr, 0, 10)
│
│       // Concatenate
│       combined = array.concat(arr1, arr2)
│       ```
│
└─ ⚠️ Array performance considerations
    ├─ Size limits
    │   - Up to 100,000 elements
    │   - Use reasonably; avoid oversized arrays
    │
    ├─ Type consistency
    │   - Array stores a single type
    │   - Type is set at declaration
    │
    └─ Loop optimization
        ```pine
        // ❌ Inefficient
        result = 0.0
        for i = 0 to array.size(arr) - 1
            result += array.get(arr, i)

        // ✅ Efficient
        result = array.sum(arr)
        ```
```

## 🗺️ Maps (key-value) - Pine Script v6

```
┌─ Choose: Map - key-value storage
│
├─ 🎯 Use cases?
│   │
│   ├─ Configuration management
│   │   └─ ✅ **Parameter config map**
│   │       ```pine
│   │       // Strategy parameter configuration
│   │       var map<string, float> config = map.new<string, float>()
│   │       if na(map.get(config, "risk"))
│   │           map.put(config, "risk", 2.0)
│   │           map.put(config, "reward", 4.0)
│   │           map.put(config, "maxDrawdown", 10.0)
│   │       ```
│   │
│   ├─ Cache computed results
│   │   └─ ✅ **Computation cache**
│   │       ```pine
│   │       // Cache RSI computation results
│   │       var map<string, float> rsiCache = map.new<string, float>()
│   │       cacheKey = str.tostring(bar_index) + "_" + str.tostring(rsiLength)
│   │       cachedRSI = map.get(rsiCache, cacheKey)
│   │       if na(cachedRSI)
│   │           cachedRSI := ta.rsi(close, rsiLength)
│   │           map.put(rsiCache, cacheKey, cachedRSI)
│   │       ```
│   │
│   ├─ Data aggregation
│   │   └─ ✅ **Statistics**
│   │       ```pine
│   │       // Aggregate by time period
│   │       var map<string, int> volumeByHour = map.new<string, int>()
│   │       hourKey = str.tostring(hour(time))
│   │       currentVol = map.get(volumeByHour, hourKey)
│   │       map.put(volumeByHour, hourKey, nz(currentVol) + volume)
│   │       ```
│   │
│   └─ State management
│       └─ ✅ **State tracking**
│           ```pine
│           // Track different states
│           var map<string, bool> states = map.new<string, bool>()
│           map.put(states, "inPosition", strategy.position_size != 0)
│           map.put(states, "trendUp", close > ta.sma(close, 20))
│           ```
│
├─ 🔧 Map operations
│   │
│   ├─ Create and initialize
│   │   ```pine
│   │   // Empty map
│   │   m = map.new<string, int>()
│   │
│   │   // With initial values
│   │   m = map.new<string, float>()
│   │   map.put(m, "key1", 1.0)
│   │   map.put(m, "key2", 2.0)
│   │   ```
│   │
│   ├─ Add and update
│   │   ```pine
│   │   // Add key-value
│   │   map.put(m, "newKey", newValue)
│   │
│   │   // Bulk insert
│   │   for [key, value] in [["a", 1], ["b", 2], ["c", 3]]
│   │       map.put(m, key, value)
│   │   ```
│   │
│   ├─ Access data
│   │   ```pine
│   │   // Get value
│   │   value = map.get(m, "key")
│   │
│   │   // With default
│   │   value = map.get(m, "key", defaultValue)
│   │
│   │   // Check if key exists
│   │   exists = map.contains(m, "key")
│   │   ```
│   │
│   └─ Remove and clear
│       ```pine
│       // Remove key
│       map.remove(m, "key")
│
│       // Get all keys
│       keys = map.keys(m)
│
│       // Get all values
│       values = map.values(m)
│
│       // Clear map
│       map.clear(m)
│       ```
│
├─ 📊 Advanced Map usage
│   │
│   ├─ Nested Map
│   │   ```pine
│   │   // 2D data structure
│   │   var map<string, map<string, float>> nested = map.new()
│   │   innerMap = map.new<string, float>()
│   │   map.put(innerMap, "value", 100.0)
│   │   map.put(nested, "outer", innerMap)
│   │   ```
│   │
│   ├─ Map as cache
│   │   ```pine
│   │   // LRU cache (example)
│   │   var map<string, int> cache = map.new<string, int>()
│   │   var int[] accessOrder = array.new<int>()
│   │
│   │   // Cache management logic
│   │   if array.size(accessOrder) > 100
│   │       oldKey = array.shift(accessOrder)
│   │       map.remove(cache, oldKey)
│   │   ```
│   │
│   └─ Dynamic configuration
│       ```pine
│       // Read configuration dynamically
│       configMap = map.from([
│           ["period", 20],
│           ["deviation", 2.0],
│           ["showBands", true]
│       ])
│       ```
│
└─ ⚠️ Map limitations
    ├─ Key types
    │   - Only string and int keys are supported
    │   - Values can be any type
    │
    ├─ Performance considerations
    │   - Fast lookups O(1)
    │   - Suitable for many key-value pairs
    │
    └─ Memory usage
        - Uses more memory than arrays
        - Control map size sensibly
```

## 🔢 Matrices (matrix) - Pine Script v6

```
┌─ Choose: Matrix - two-dimensional data
│
├─ 🎯 Use cases?
│   │
│   ├─ Math calculations
│   │   └─ ✅ **Linear algebra operations**
│   │       ```pine
│   │       // Create matrix
│   │       m = matrix.new<float>(3, 3, 0.0)
│   │
│   │       // Fill values
│   │       for i = 0 to 2
│   │           for j = 0 to 2
│   │               matrix.set(m, i, j, i + j)
│   │
│   │       // Matrix operations
│   │       identity = matrix.identity(3)
│   │       result = matrix.mult(m, identity)
│   │       ```
│   │
│   ├─ Data grid
│   │   └─ ✅ **Two-dimensional data storage**
│   │       ```pine
│   │       // Price grid
│   │       priceGrid = matrix.new<float>(10, 10, 0.0)
│   │
│   │       // Fill price data
│   │       basePrice = close
│   │       for i = 0 to 9
│   │           for j = 0 to 9
│   │               price = basePrice * (1 + i * 0.01) * (1 + j * 0.01)
│   │               matrix.set(priceGrid, i, j, price)
│   │       ```
│   │
│   ├─ Correlation matrix
│   │   └─ ✅ **Asset correlation**
│   │       ```pine
│   │       // Multi-asset correlation matrix
│   │       corrMatrix = matrix.new<float>(5, 5, 0.0)
│   │
│   │       // Compute correlation
│   │       assets = array.from("AAPL", "GOOGL", "MSFT", "AMZN", "FB")
│   │       for i = 0 to 4
│   │           for j = 0 to 4
│   │               corr = ta.correlation(returns[i], returns[j], 20)
│   │               matrix.set(corrMatrix, i, j, corr)
│   │       ```
│   │
│   └─ Machine learning data
│       └─ ✅ **Feature matrix**
│           ```pine
│           // Feature data matrix
│           features = matrix.new<float>(100, 5, 0.0)
│
│           // Features: RSI, MACD, MA diff, etc.
│           for i = 0 to 99
│               matrix.set(features, i, 0, ta.rsi(close[i], 14))
│               matrix.set(features, i, 1, ta.macd(close[i])[0])
│               matrix.set(features, i, 2, close[i] - ta.sma(close[i], 20))
│           ```
│
├─ 🔧 Matrix operations
│   │
│   ├─ Basic operations
│   │   ```pine
│   │   // Create matrix
│   │   m = matrix.new<int>(rows, cols, initialValue)
│   │
│   │   // Get element
│   │   value = matrix.get(m, row, col)
│   │
│   │   // Set element
│   │   matrix.set(m, row, col, value)
│   │
│   │   // Get dimensions
│   │   rows = matrix.rows(m)
│   │   cols = matrix.columns(m)
│   │   ```
│   │
│   ├─ Matrix operations
│   │   ```pine
│   │   // Addition
│   │   result = matrix.add(m1, m2)
│   │
│   │   // Subtraction
│   │   result = matrix.sub(m1, m2)
│   │
│   │   // Multiplication
│   │   result = matrix.mult(m1, m2)
│   │
│   │   // Dot product
│   │   result = matrix.dot(m1, m2)
│   │
│   │   // Transpose
│   │   transposed = matrix.transpose(m)
│   │   ```
│   │
│   ├─ Special matrices
│   │   ```pine
│   │   // Identity matrix
│   │   identity = matrix.identity(size)
│   │
│   │   // Zero matrix
│   │   zeros = matrix.new<float>(rows, cols, 0.0)
│   │
│   │   // Diagonal matrix
│   │       diagonal = matrix.diagonal(array.from(1, 2, 3, 4))
│   │   ```
│   │
│   └─ Advanced operations
│       ```pine
│       // Determinant
│       det = matrix.det(m)

│       // Inverse
│       inverse = matrix.inv(m)

│       // Pseudo-inverse
│       pseudoInv = matrix.pinv(m)

│       // Eigenvalues (requires library)
│       ```
│
├─ 📊 Matrix tips
│   │
│   ├─ Data conversion
│   │   ```pine
│   │   // Array to Matrix
│   │   arr = array.from(1, 2, 3, 4, 5, 6)
│   │   m = matrix.from_array(arr, 2, 3)  // 2 rows x 3 cols
│   │
│   │   // Matrix to Array
│   │   flatArr = matrix.to_array(m)
│   │   ```
│   │
│   ├─ Row/column operations
│   │   ```pine
│   │   // Get row
│   │   row = matrix.row(m, rowIndex)

│   │   // Get column
│   │   col = matrix.col(m, colIndex)

│   │   // Set row
│   │   matrix.set_row(m, rowIndex, newRowArray)

│   │   // Set column
│   │   matrix.set_col(m, colIndex, newColArray)
│   │   ```
│   │
│   └─ Submatrices
│       ```pine
│       // Extract submatrix
│       subM = matrix.submatrix(m, startRow, endRow, startCol, endCol)

│       // Concatenate matrices
│       combined = matrix.concat_vert(m1, m2)  // vertical concat
│       combined = matrix.concat_horiz(m1, m2)  // horizontal concat
│       ```
│
└─ ⚠️ Matrix limitations
    ├─ Dimension limits
    │   - Max dimensions limited by memory
    │   - Control matrix size reasonably
    │
    ├─ Computational complexity
    │   - Matrix multiplication O(n^3)
    │   - Mind performance impact
    │
    └─ Data types
        - All elements must be the same type
        - Supports int, float, bool
```

## 🔄 Data Structure Comparison and Selection

```
┌─ Data structure selection guide
│
├─ 📊 Choose by data dimensionality
│   │
│   ├─ One-dimensional list → Array
│   │   - Time series data
│   │   - Simple collections of values
│   │   - Historical price storage
│   │
│   ├─ Key-value pairs → Map
│   │   - Configuration parameters
│   │   - Caching systems
│   │   - Dictionary-style data
│   │
│   └─ Two-dimensional table → Matrix
│       - Math operations
│       - Grid data
│       - Feature matrices
│
├─ ⚡ Choose by performance needs
│   │
│   ├─ Fast lookup → Map
│   │   - O(1) lookup time
│   │   - Many key-value pairs
│   │
│   ├─ Sequential access → Array
│   │   - O(1) index access
│   │   - Iteration
│   │
│   └─ Mathematical operations → Matrix
│       - Linear algebra
│       - Batch computation
│
├─ 💾 Choose by memory usage
│   │
│   ├─ Memory efficient → Array
│   │   - Minimal memory footprint
│   │   - Simple data
│   │
│   ├─ Flexible features → Map
│   │   - Dynamic keys
│   │   - Metadata storage
│   │
│   └─ Structured data → Matrix
│       - Two-dimensional data
│       - Tabular structure
│
└─ 🔄 Conversions
    ├─ Array ↔ Map
    │   ```pine
    │   // Array to Map
    │   arr = array.from(1, 2, 3)
    │   m = map.new<string, int>()
    │   for i = 0 to array.size(arr) - 1
    │     map.put(m, "key" + str.tostring(i), arr[i])
    │   ```
    │
    ├─ Array ↔ Matrix
    │   ```pine
    │   // 1D Array to Matrix
    │   m = matrix.from_array(arr, rows, cols)
    │
    │   // Matrix to Array
    │   arr = matrix.to_array(m)
    │   ```
    │
    └─ Map → Array
        ```pine
        // Map keys to Array
        keys = map.keys(m)

        // Map values to Array
        values = map.values(m)
        ```
```

## 📝 Best Practices Summary

### Array best practices
```pine
// 1. Pre-allocate size
var float[] prices = array.new<float>(100, 0.0)

// 2. Use built-ins
sum = array.sum(arr)  // instead of manual loops

// 3. Bulk operations
array.fill(arr, value)  // bulk fill

// 4. Type safety
var int[] numbers = array.new<int>()  // explicit typing
```

### Map best practices
```pine
// 1. Use descriptive keys
map.put(config, "stopLossPercent", 2.0)

// 2. Check key existence
if map.contains(cache, key)
    value = map.get(cache, key)

// 3. Use default values
value = map.get(m, "key", defaultValue)

// 4. Cache management
if map.size(cache) > maxCacheSize
    map.clear(cache)
```

### Matrix best practices
```pine
// 1. Pre-compute dimensions
rows = math.ceil(math.sqrt(dataSize))
cols = math.ceil(dataSize / rows)

// 2. Prefer matrix ops
result = matrix.mult(m1, m2)  // rather than nested loops

// 3. Special matrices
identity = matrix.identity(size)  // use built-in

// 4. Memory management
if matrix.rows(m) > maxRows
    m = matrix.submatrix(m, 0, maxRows, 0, matrix.columns(m))
```

## ⚠️ Common mistakes

1. **Array index out of bounds**
   ```pine
   // ❌ Wrong
   value = arr[array.size(arr)]  // last index is size-1

   // ✅ Correct
   value = arr[array.size(arr) - 1]
   ```

2. **Map type error**
   ```pine
   // ❌ Wrong: float cannot be used as a key
   m = map.new<float, string>()

   // ✅ Correct
   m = map.new<string, float>()
   ```

3. **Matrix dimension mismatch**
   ```pine
   // ❌ Wrong: mismatched dimensions
   result = matrix.mult(m2x3, m4x5)

   // ✅ Correct
   result = matrix.mult(m2x3, m3x5)
   ```

4. **Memory leak**
   ```pine
   // ❌ Wrong: unbounded growth
   array.push(arr, value)  // never pruned

   // ✅ Correct: limit size
   if array.size(arr) > maxSize
       array.shift(arr)
   ```

## 🎯 Quick decision table

| Need | Array | Map | Matrix |
|------|-------|-----|---------|
| Time series data | ✅ | ❌ | ❌ |
| Key lookup | ❌ | ✅ | ❌ |
| Matrix math | ❌ | ❌ | ✅ |
| Simple value list | ✅ | ❌ | ❌ |
| Configuration parameters | ❌ | ✅ | ❌ |
| 2D data grid | Possible | Possible | ✅ |
| Caching system | ❌ | ✅ | ❌ |
| Performance (lookup) | O(n) | O(1) | O(n·m) |
| Memory efficiency | High | Medium | Low |

The key to choosing a data structure: **match data characteristics, optimize access patterns, and consider performance constraints**.
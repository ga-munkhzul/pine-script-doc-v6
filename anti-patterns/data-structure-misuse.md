# Data Structure Misuse Anti-Patterns

Pine Script v6 provides data structures such as arrays, maps, and matrices. Misusing these structures can cause errors and performance issues.

## 1. Array out-of-bounds errors

### ❌ Incorrect example: array index out of bounds
```pine
//@version=6
indicator("Error: Array out of bounds")

// ❌ Attempt to access a non-existent index
var float[] prices = array.new<float>()
array.push(prices, close)
array.push(prices, close[1])

// Error: the array has only 2 elements but tries to access index 5
value = array.get(prices, 5)  // Runtime error!
plot(value)
```

### 🚨 Problem explanation
- Pine Script arrays are 0-indexed
- Accessing an index beyond `size - 1` causes a runtime error
- The error stops the script

### ✅ Correct approach: check index validity
```pine
//@version=6
indicator("Correct: Check index")

var float[] prices = array.new<float>()
array.push(prices, close)
array.push(prices, close[1])

// ✅ Check whether index is valid
index = 5
value = index < array.size(prices) ? array.get(prices, index) : na
plot(value, title="Value", display=display.data_window)
```

## 2. Type mismatch

### ❌ Incorrect example: mixed-type array
```pine
//@version=6
indicator("Error: Type mismatch")

// ❌ Declared as a float array but trying to insert int
var float[] mixedArray = array.new<float>()

array.push(mixedArray, close)      // OK, float
array.push(mixedArray, 10)        // OK, int can be auto-converted to float
array.push(mixedArray, "text")    // Error! cannot insert string
```

### 🚨 Problem explanation
- Pine Script arrays can store only one type
- The type is determined at declaration
- Inserting an incompatible type causes a compile-time error

### ✅ Correct approach: unify types or use different arrays
```pine
//@version=6
indicator("Correct: Unified types")

// ✅ Method 1: use float consistently
var float[] numbers = array.new<float>()
array.push(numbers, close)
array.push(numbers, 10.0)  // Explicit conversion to float

// ✅ Method 2: create separate arrays for different types
var string[] texts = array.new<string>()
var float[] values = array.new<float>()

array.push(texts, "Price")
array.push(values, close)
```

## 3. Modifying arrays during iteration

### ❌ Incorrect example: modifying array while iterating
```pine
//@version=6
indicator("Error: Modify while iterating")

var float[] data = array.from(1, 2, 3, 4, 5)

// ❌ Adding elements while iterating causes infinite loop
for i = 0 to array.size(data) - 1
    value = array.get(data, i)
    if value > 3
        array.push(data, value * 2)  // Dangerous! array is growing
```

### 🚨 Problem explanation
- Modifying array size while iterating leads to undefined behavior
- May cause infinite loops or skip elements
- Pine Script may limit loop counts (100)

### ✅ Correct approach: use a temporary array
```pine
//@version=6
indicator("Correct: Use temporary array")

var float[] data = array.from(1, 2, 3, 4, 5)
var float[] newData = array.new<float>()

// ✅ Collect elements to add
for i = 0 to array.size(data) - 1
    value = array.get(data, i)
    if value > 3
        array.push(newData, value * 2)

// Merge after iteration completes
for i = 0 to array.size(newData) - 1
    array.push(data, array.get(newData, i))
```

## 4. Map key type errors

### ❌ Incorrect example: using invalid key types
```pine
//@version=6
indicator("Error: Map key type error")

// ❌ Map supports only string or int as keys
var map<float, string> wrongMap = map.new<float, string>()
map.put(wrongMap, 1.5, "value")  // Error! float cannot be a key

// ❌ Trying to use bool as a key
var map<bool, float> boolMap = map.new<bool, float>()
map.put(boolMap, true, 100.0)     // Error! bool cannot be a key
```

### 🚨 Problem explanation
- Pine Script v6 Maps support only `string` and `int` as keys
- This is a Pine Script limitation
- Using other types causes compile-time errors

### ✅ Correct approach: use supported key types
```pine
//@version=6
indicator("Correct: Supported key types")

// ✅ Use string as key
var map<string, float> config = map.new<string, float>()
map.put(config, "risk", 2.0)
map.put(config, "reward", 4.0)

// ✅ Use int as key
var map<int, string> levels = map.new<int, string>()
map.put(levels, 1, "Support")
map.put(levels, 2, "Resistance")

// If you need other types, convert to string
key = str.tostring(someValue)
```

## 5. Matrix dimension mismatch

### ❌ Incorrect example: wrong matrix dimensions
```pine
//@version=6
indicator("Error: Matrix dimension error")

// Create matrices of different dimensions
m1 = matrix.new<float>(2, 3)  // 2 rows, 3 columns
m2 = matrix.new<float>(4, 5)  // 4 rows, 5 columns

// ❌ Try to multiply different dimensions
result = matrix.mult(m1, m2)  // Error! Dimension mismatch

// ❌ Out-of-range when setting element
matrix.set(m1, 5, 5, 100.0)  // Error! Index out of range
```

### 🚨 Problem explanation
- Matrix multiplication requires the first matrix's columns to equal the second matrix's rows
- Matrix element access must be within valid ranges
- Dimension errors cause runtime errors

### ✅ Correct approach: match dimensions
```pine
//@version=6
indicator("Correct: Matrix dimension match")

// ✅ Create matrices with matching dimensions
m1 = matrix.new<float>(2, 3)  // 2 rows, 3 columns
m2 = matrix.new<float>(3, 4)  // 3 rows, 4 columns

// Multiplication valid: 3 columns = 3 rows
result = matrix.mult(m1, m2)  // Result is 2 rows x 4 columns

// ✅ Safely set elements
rows = matrix.rows(m1)
cols = matrix.columns(m1)
if rows > 0 and cols > 0
    matrix.set(m1, 0, 0, 100.0)  // Safe access
```

## 6. Inefficient array operations

### ❌ Incorrect example: frequent array operations
```pine
//@version=6
indicator("Error: Inefficient array operations")

var float[] data = array.new<float>()

// ❌ Frequently inserting at the beginning (O(n) operation)
for i = 0 to 100
    array.unshift(data, close[i])  // Must move all elements each time

// ❌ Frequently removing middle elements
for i = 50 to 60
    array.remove(data, i)  // Must move subsequent elements each time
```

### 🚨 Problem explanation
- `array.unshift()` requires moving all elements when inserting at the start
- `array.remove()` requires moving subsequent elements when removing in the middle
- Performing these frequently is poor performance

### ✅ Correct approach: choose appropriate operations
```pine
//@version=6
indicator("Correct: Efficient array operations")

var float[] data = array.new<float>()

// ✅ Use push to add at the end (O(1) operation)
for i = 0 to 100
    array.push(data, close[i])

// ✅ If you need to reverse, do it once afterwards
if array.size(data) > 0
    array.reverse(data)

// ✅ Delete in batches instead of one-by-one
if array.size(data) > 50
    data = array.slice(data, 0, 50)
```

## 7. Misuse of Map

### ❌ Incorrect example: using Map for a simple list
```pine
//@version=6
indicator("Error: Map misuse")

// ❌ Using Map to store a simple sequence
var map<string, float> priceMap = map.new<string, float>()
map.put(priceMap, "0", close[0])
map.put(priceMap, "1", close[1])
map.put(priceMap, "2", close[2])

// Access requires string conversion, inefficient
for i = 0 to 2
    price = map.get(priceMap, str.tostring(i))
```

### 🚨 Problem explanation
- Map is suitable for key-value lookups
- For simple indexed sequences, Array is more appropriate
- String-key operations on Map are slower than array indexing

### ✅ Correct approach: choose the right data structure
```pine
//@version=6
indicator("Correct: Choose appropriate structure")

// ✅ Use array for simple sequences
var float[] prices = array.new<float>()
array.push(prices, close)
array.push(prices, close[1])
array.push(prices, close[2])

// ✅ Use Map where key-value lookups are truly needed
var map<string, float> config = map.new<string, float>()
map.put(config, "stopLoss", 2.0)
map.put(config, "takeProfit", 4.0)
stopLoss = map.get(config, "stopLoss")
```

## 8. Using Matrix for simple data

### ❌ Incorrect example: overusing Matrix
```pine
//@version=6
indicator("Error: Overusing Matrix")

// ❌ Using a matrix when only a few values need to be stored
data = matrix.new<float>(1, 5)
matrix.set(data, 0, 0, close)
matrix.set(data, 0, 1, high)
matrix.set(data, 0, 2, low)
matrix.set(data, 0, 3, open)
matrix.set(data, 0, 4, volume)

// Access becomes complex
closeValue = matrix.get(data, 0, 0)
```

### 🚨 Problem explanation
- Matrix is suitable for 2D mathematical operations
- Using Matrix for simple data is overly complex
- Access and operations are less intuitive than arrays

### ✅ Correct approach: use arrays or multiple variables
```pine
//@version=6
indicator("Correct: Simple structures")

// ✅ Method 1: use an array
var float[] ohlcv = array.new<float>()
array.push(ohlcv, close)
array.push(ohlcv, high)
array.push(ohlcv, low)
array.push(ohlcv, open)
array.push(ohlcv, volume)

// ✅ Method 2: use a structured approach (if data is related)
type OHLCV
    float close
    float high
    float low
    float open
    float volume

data = OHLCV.new(close, high, low, open, volume)
```

## 9. Ignoring type checks

### ❌ Incorrect example: not checking type conversions
```pine
//@version=6
indicator("Error: Ignoring type checks")

// ❌ Conversion that may fail
strValue = "123.45"
numValue = float(strValue)  // Errors if strValue is not a valid number

// ❌ Not checking na values
value = someCalculation()
result = value * 2  // If value is na, this causes an error
```

### 🚨 Problem explanation
- Pine Script does not automatically handle all type conversion errors
- Operations with na values produce incorrect results
- Explicit checks and handling are required

### ✅ Correct approach: type safety and na checks
```pine
//@version=6
indicator("Correct: Type safety")

// ✅ Safe string conversion
strValue = "123.45"
numValue = str.tonumber(strValue)
result = na(numValue) ? 0.0 : numValue

// ✅ Check for na
value = someCalculation()
if not na(value)
    result = value * 2
else
    result = 0.0

// ✅ Use nz function
result = nz(value, 0.0) * 2
```

## 10. Wrong choice of Array/Map/Matrix

### ❌ Incorrect example: choosing the wrong data structure
```pine
//@version=6
indicator("Error: Wrong structure choice")

// ❌ Uses array when fast lookup is needed
var float[] searchArray = array.from(10, 20, 30, 40, 50)
target = 30
found = false
for i = 0 to array.size(searchArray) - 1  // O(n) lookup
    if array.get(searchArray, i) == target
        found := true
        break
```

### 🚨 Problem explanation
- Array search is O(n)
- Map lookup is O(1)
- Choosing the wrong structure impacts performance

### ✅ Correct approach: choose based on requirements
```pine
//@version=6
indicator("Correct: Structure selection")

// ✅ Use Map for fast lookups
var map<int, bool> lookupMap = map.new<int, bool>()
map.put(lookupMap, 10, true)
map.put(lookupMap, 20, true)
map.put(lookupMap, 30, true)

target = 30
found = map.contains(lookupMap, target)  // O(1) lookup

// ✅ Use array for sequential access
var float[] sequence = array.from(10, 20, 30, 40, 50)
for i = 0 to array.size(sequence) - 1
    value = array.get(sequence, i)
    // Process the sequence
```

## Data structure selection guide

| Requirement | Best choice | Alternative | Notes |
|-------------|------------|-------------|-------|
| Simple list | Array | - | Fast index access |
| Key-value lookup | Map | Array (small datasets) | Map supports only string/int keys |
| Math matrix | Matrix | Array of Arrays | Mind the dimension match |
| Fixed-size set | Array | Map | Preallocate size |
| Dynamic growth | Array | - | Limit max size |
| Cached data | Map/Array | - | Consider cleanup strategy |

## Best practices summary

1. **Always check bounds**: validate index before array access
2. **Type consistency**: keep array element types consistent
3. **Choose appropriate structure**: select Array/Map/Matrix by need
4. **Avoid modifying while iterating**: use a temporary array
5. **Mind performance**: know the complexity of operations
6. **Handle na values**: use nz() or explicit checks
7. **Preallocate size**: when the size is known
8. **Batch operations**: use built-ins instead of manual loops

Remember: **Choosing the right data structure is the foundation of efficient code!**
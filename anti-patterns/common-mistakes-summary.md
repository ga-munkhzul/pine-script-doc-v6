# Pine Script 常见错误总结

## 📊 错误统计

以下是 Pine Script 开发中最容易犯的错误，按发生频率排序：

| 错误类型 | 发生频率 | 影响等级 | 容易发现 |
|---------|---------|---------|---------|
| 重绘错误 | 极高 | 严重 | 困难 |
| 性能问题 | 高 | 中等 | 中等 |
| 逻辑错误 | 高 | 严重 | 困难 |
| 数据结构误用 | 中等 | 中等 | 容易 |
| 类型错误 | 低 | 严重 | 容易 |

## 🚫 十大致命错误

### 1. **未来泄漏** (最严重)
```pine
// ❌ 致命错误
dailyClose = request.security(syminfo.tickerid, "D", close)

// ✅ 正确做法
dailyClose = request.security(syminfo.tickerid, "D", close[1])
```

### 2. **实时数据误用**
```pine
// ❌ 会导致重绘
if ta.crossover(close, ta.sma(close, 20))
    strategy.entry("Long", strategy.long)

// ✅ 等待确认
if ta.crossover(close, ta.sma(close, 20)) and barstate.isconfirmed
    strategy.entry("Long", strategy.long)
```

### 3. **无限循环**
```pine
// ❌ 危险：可能无限循环
var float value = 0
value := ta.sma(value, 10)  // 循环引用

// ✅ 正确计算
rawValue = close * 0.1
value := ta.sma(rawValue, 10)
```

### 4. **数组越界**
```pine
// ❌ 运行时错误
value = array.get(arr, array.size(arr))

// ✅ 安全访问
index = array.size(arr) - 1
value = index >= 0 ? array.get(arr, index) : na
```

### 5. **内存泄漏**
```pine
// ❌ 无限增长
var float[] data = array.new<float>()
array.push(data, close)  // 永不清理

// ✅ 限制大小
if array.size(data) > 100
    array.shift(data)
```

### 6. **条件顺序错误**
```pine
// ❌ 第二个条件永远不执行
if close > ma50
    // 代码
else if close > ma20  // 永远false
    // 代码
```

### 7. **忽略na值**
```pine
// ❌ na值传播
result = (naValue + 10) / 2  // 结果是na

// ✅ 处理na值
result = (nz(naValue, 0) + 10) / 2
```

### 8. **性能陷阱**
```pine
// ❌ 重复计算
for i = 0 to 100
    ma = ta.sma(close, 20)  // 每次都计算

// ✅ 预计算
ma = ta.sma(close, 20)
for i = 0 to 100
    // 使用ma
```

### 9. **状态不同步**
```pine
// ❌ 多个状态变量
inLong1 = strategy.position_size > 0
inLong2 = someOtherCondition  // 可能不一致

// ✅ 单一状态
inLong = strategy.position_size > 0
```

### 10. **时间框架混淆**
```pine
// ❌ 错误认为这是小时数据
h1Trend = close > ta.sma(close, 50)  // 实际是当前时间框架

// ✅ 明确获取
h1Trend = request.security(..., "60", close) > request.security(..., "60", ta.sma(close, 50))
```

## 🛡️ 防错原则

### 1. **防御性编程**
```pine
// 总是检查边界
if index >= 0 and index < array.size(arr)
    value = array.get(arr, index)
```

### 2. **明确性原则**
```pine
// 使用括号明确运算顺序
condition = (rsi > 50 and close > ma) or volume > avgVolume
```

### 3. **单一职责**
```pine
// 每个变量/函数只做一件事
var bool inPosition = strategy.position_size > 0  // 状态跟踪
```

### 4. **最小化原则**
```pine
// 只在需要时计算
if showIndicator
    expensiveCalculation()
```

### 5. **一致性原则**
```pine
// 保持命名和风格一致
ma20 = ta.sma(close, 20)
ma50 = ta.sma(close, 50)  // 一致的命名
```

## 📝 Code review checklist

### ✅ Pre-commit checks

1. Repainting checks
   - [ ] Is there an offset in request.security()?
   - [ ] Do you wait for barstate.isconfirmed?
   - [ ] Do you use timenow for historical checks?

2. Performance checks
   - [ ] Do loops perform repeated calculations?
   - [ ] Do arrays grow without bound?
   - [ ] Are request.security() calls reasonable?

3. Logic checks
   - [ ] Is the condition order correct?
   - [ ] Is state consistent?
   - [ ] Are all edge cases handled?

4. Type checks
   - [ ] Any type casts?
   - [ ] Are array types consistent?
   - [ ] Are na values handled?

5. Test checks
   - [ ] Tested on different timeframes?
   - [ ] How does it perform historically?
   - [ ] Does real-time behavior meet expectations?

## 🎯 Quick fix templates

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

## 💡 Remember these

1. If backtests look too perfect, there's probably repainting
2. Performance issues often come from loops and arrays
3. Logic errors are hardest to spot; test more
4. Type errors are easiest; the compiler will help
5. Good code = Simple + Clear + Testable

## 🚀 进阶建议

1. **使用版本控制**
   - 每个重大修改提交
   - 可以回滚到工作版本

2. **编写测试用例**
   - 正常情况
   - 边界情况
   - 异常情况

3. **添加注释**
   ```pine
   // 为什么这样做，而不是做什么
   // 使用偏移避免未来泄漏
   dailyClose = request.security(..., close[1])
   ```

4. **模块化代码**
   - 每个函数一个职责
   - 可复用的功能提取出来

5. **持续学习**
   - 查看他人的代码
   - 学习最佳实践
   - 保持更新知识

---

**最终建议**：代码不仅要能运行，更要正确、高效、可维护。多思考，多测试，多改进！
# Pine Script v6 Concept Index

## Table of Contents
- [Core Concepts](#core-concepts)
- [Quick Lookup by Category](#quick-lookup-by-category)
- [Learning Paths](#learning-paths)
- [Quick Reference](#quick-reference)
- [Related Resources](#related-resources)
- [Search Tips](#search-tips)

---

## Core Concepts

### A
- **Alerts** — [📖 Docs](pine-script-docs/concepts/alerts.md) | [❌ Anti-patterns](anti-patterns/alert-mistakes.md) | [🌳 Decision Tree](decision-trees/alert-implementation.md)
- **Arrays** — [📖 Docs](pine-script-docs/language/arrays.md) | [❌ Anti-patterns](anti-patterns/data-structure-misuse.md) | [🌳 Decision Tree](decision-trees/data-structure-selection.md)
- **Annotations** — [📖 Reference](pine-script-reference/Annotations.md)

### B
- **Bar States** — [📖 Docs](pine-script-docs/concepts/bar-states.md)
- **Built-ins** — [📖 Docs](pine-script-docs/language/built-ins.md)

### C
- **Chart Information** — [📖 Docs](pine-script-docs/concepts/chart-information.md)
- **Conditional Structures** — [📖 Docs](pine-script-docs/language/conditional-structures.md) | [❌ Anti-patterns](anti-patterns/logic-errors.md)
- **Constants** — [📖 Reference](pine-script-reference/Constants.md)
- **Concepts** — [📖 Docs](pine-script-docs/concepts/)

### D
- **Data Structures** — [🌳 Decision Tree](decision-trees/data-structure-selection.md) | [❌ Anti-patterns](anti-patterns/data-structure-misuse.md)

### E
- **Enums** — [📖 Docs](pine-script-docs/language/enums.md)
- **Execution Model** — [📖 Docs](pine-script-docs/language/execution-model.md)

### F
- **Functions** — [📖 Docs](pine-script-docs/language/user-defined-functions.md) | [📖 Reference](pine-script-reference/Functions.md)

### I
- **Identifiers** — [📖 Docs](pine-script-docs/language/identifiers.md)
- **Inputs** — [📖 Docs](pine-script-docs/concepts/inputs.md)
- **Indicator** — [🌳 Decision Tree](decision-trees/script-type-selection.md)

### L
- **Language** — [📖 Docs](pine-script-docs/language/)
- **Libraries** — [📖 Docs](pine-script-docs/concepts/libraries.md)
- **Loops** — [📖 Docs](pine-script-docs/language/loops.md) | [❌ Anti-patterns](anti-patterns/performance-traps.md)

### M
- **Maps** — [🌳 Decision Tree](decision-trees/data-structure-selection.md) | [❌ Anti-patterns](anti-patterns/data-structure-misuse.md)
- **Matrices** — [🌳 Decision Tree](decision-trees/data-structure-selection.md) | [❌ Anti-patterns](anti-patterns/data-structure-misuse.md)
- **Methods** — [📖 Docs](pine-script-docs/language/methods.md)

### O
- **Objects** — [📖 Docs](pine-script-docs/language/objects.md)
- **Operators** — [📖 Docs](pine-script-docs/language/operators.md) | [📖 Reference](pine-script-reference/Operators.md)

### P
- **Performance** — [❌ Anti-patterns](anti-patterns/performance-traps.md) | [🌳 Decision Tree](decision-trees/performance-optimization.md)
- **Plotting** — [📖 Docs](pine-script-docs/visuals/plots.md) | [🌳 Decision Tree](decision-trees/visualization-selection.md)
- **Properties** — [📖 Docs](pine-script-docs/visuals/overview.md)

### R
- **Repainting** — [📖 Docs](pine-script-docs/concepts/repainting.md) | [❌ Anti-patterns](anti-patterns/repainting-errors.md) | [🌳 Decision Tree](decision-trees/avoid-repainting.md)
- **Reference** — [📖 Reference](pine-script-reference/)
- **Request Security** — [🌳 Decision Tree](decision-trees/multi-timeframe.md) | [❌ Anti-patterns](anti-patterns/multi-timeframe-errors.md)

### S
- **Script Structure** — [📖 Docs](pine-script-docs/language/script-structure.md)
- **Sessions** — [📖 Docs](pine-script-docs/concepts/sessions.md)
- **Strategies** — [📖 Docs](pine-script-docs/concepts/strategies.md) | [🌳 Decision Tree](decision-trees/script-type-selection.md) | [❌ Anti-patterns](anti-patterns/strategy-mistakes.md)
- **Strings** — [📖 Docs](pine-script-docs/concepts/strings.md)

### T
- **Time** — [📖 Docs](pine-script-docs/concepts/time.md)
- **Timeframes** — [📖 Docs](pine-script-docs/concepts/timeframes.md)
- **Type System** — [📖 Docs](pine-script-docs/language/type-system.md) | [📖 Reference](pine-script-reference/Types.md)

### V
- **Variable Declarations** — [📖 Docs](pine-script-docs/language/variable-declarations.md) | [📖 Reference](pine-script-reference/Variables.md)
- **Variables** — [📖 Docs](pine-script-docs/language/variable-declarations.md)
- **Visuals** — [📖 Docs](pine-script-docs/visuals/) | [🌳 Decision Tree](decision-trees/visualization-selection.md)

---

## Quick Lookup by Category

### 🔧 Language Basics
- [Execution Model](pine-script-docs/language/execution-model.md) — How Pine Script runs
- [Type System](pine-script-docs/language/type-system.md) — Types and qualifiers
- [Script Structure](pine-script-docs/language/script-structure.md) — Script organization
- [Variable Declarations](pine-script-docs/language/variable-declarations.md) — var, varip, simple, etc.
- [Identifiers](pine-script-docs/language/identifiers.md) — Naming rules

### 📊 Data Handling
- [Arrays](pine-script-docs/language/arrays.md) — One-dimensional collections
- [Maps](knowledge-graph.json#maps) — Key–value storage (v6)
- [Matrices](knowledge-graph.json#matrices) — Two-dimensional data (v6)
- [Loops](pine-script-docs/language/loops.md) — Iterative processing
- [Built-ins](pine-script-docs/language/built-ins.md) — open, high, low, close, etc.

### 🎯 Control Flow
- [Conditional Structures](pine-script-docs/language/conditional-structures.md) — if, switch
- [Loops](pine-script-docs/language/loops.md) — for, while
- [Operators](pine-script-docs/language/operators.md) — arithmetic, comparison, logic
- [Functions](pine-script-docs/language/user-defined-functions.md) — user-defined functions

### 📈 Script Types
- [Indicator](decision-trees/script-type-selection.md#indicator) — Analysis tools
- [Strategy](decision-trees/script-type-selection.md#strategy) — Trading simulation
- [Library](decision-trees/script-type-selection.md#library) — Code reuse

### 🎨 Visualization
- [Plots](pine-script-docs/visuals/plots.md) — Basic plotting
- [Backgrounds](pine-script-docs/visuals/backgrounds.md) — Background color
- [Lines and Boxes](pine-script-docs/visuals/lines-and-boxes.md) — Geometric shapes
- [Tables](pine-script-docs/visuals/tables.md) — Data tables
- [Text and Shapes](pine-script-docs/visuals/text-and-shapes.md) — Text and graphics

### ⚠️ Common Issues
- [Repainting](anti-patterns/repainting-errors.md) — Avoid repainting issues
- [Performance](anti-patterns/performance-traps.md) — Optimize script performance
- [Logic Errors](anti-patterns/logic-errors.md) — Avoid logic pitfalls
- [Data Structure Misuse](anti-patterns/data-structure-misuse.md) — Use data structures correctly

---

## Learning Paths

### 🌱 Beginner Path (1–2 weeks)
1. [Script Structure](pine-script-docs/language/script-structure.md) — Understand the basics
2. [Type System Basics](pine-script-docs/language/type-system.md#types) — Basic types
3. [Variable Declarations](pine-script-docs/language/variable-declarations.md) — Store data
4. [Conditional Structures](pine-script-docs/language/conditional-structures.md) — Control logic
5. [Built-ins](pine-script-docs/language/built-ins.md#built-in-variables) — Use OHLCV
6. [Basic Plotting](pine-script-docs/visuals/plots.md) — Display data
7. [Inputs](pine-script-docs/concepts/inputs.md) — User interaction
8. [Create Your First Indicator](decision-trees/script-type-selection.md#indicator) — Practice

### 🌿 Intermediate Path (2–4 weeks)
1. [Execution Model](pine-script-docs/language/execution-model.md) — Deep understanding
2. [Time Series](pine-script-docs/language/execution-model.md#time-series) — Historical data
3. [Arrays](pine-script-docs/language/arrays.md) — Data collections
4. [Functions](pine-script-docs/language/user-defined-functions.md) — Code reuse
5. [Loops](pine-script-docs/language/loops.md) — Iterative computation
6. [Alerts](pine-script-docs/concepts/alerts.md) — Notification system
7. [Strategy Basics](pine-script-docs/concepts/strategies.md) — Trading logic
8. [Avoid Repainting](decision-trees/avoid-repainting.md) — Code stability

### 🌳 Advanced Path (1–2 months)
1. [Type System — Advanced](pine-script-docs/language/type-system.md#qualifiers) — Qualifiers
2. [Maps and Matrices](decision-trees/data-structure-selection.md) — New in v6
3. [Library Development](pine-script-docs/concepts/libraries.md) — Code organization
4. [Multi-timeframe Data](decision-trees/multi-timeframe.md) — Multiple timeframes
5. [Performance Optimization](decision-trees/performance-optimization.md) — Improve efficiency
6. [Advanced Visualization](decision-trees/visualization-selection.md) — Professional charts
7. [Anti-patterns](anti-patterns/) — Avoid mistakes
8. [Knowledge Graph](knowledge-graph.json) — Concept relationships

---

## Quick Reference

### Common Function Cheatsheet

#### Technical indicators
```pine
// Moving averages
ta.sma(source, length)
ta.ema(source, length)
ta.wma(source, length)

// RSI
ta.rsi(source, length)

// MACD
[macd, signal, hist] = ta.macd(source, 12, 26, 9)

// ATR
ta.atr(length)
```

#### Array operations
```pine
// Create
arr = array.new<float>(size, initialValue)
arr = array.from(val1, val2, val3)

// Append
array.push(arr, value)
array.unshift(arr, value)

// Access
value = array.get(arr, index)
value = arr[index]  // v6+

// Statistics
sum = array.sum(arr)
avg = array.avg(arr)
max = array.max(arr)
```

#### Map operations (v6)
```pine
// Create
m = map.new<string, float>()

// Put
map.put(m, "key", value)

// Get
value = map.get(m, "key", defaultValue)

// Check
exists = map.contains(m, "key")
```

### ⚡ Performance Tips

- ✅ Use `var` to cache results
- ✅ Limit array sizes
- ✅ Prefer built-ins over loops
- ✅ Cache `request.security()` calls
- ❌ Avoid deep nested loops
- ❌ Don't recompute inside loops

### 🚨 Repainting Checklist

- ✅ Use `[1]` to offset historical data
- ✅ Use `barstate.isconfirmed`
- ✅ Set `lookahead=barmerge.lookahead_on`
- ❌ Don't use `close/high/low` on realtime bars
- ❌ Don't use `timenow` for historical decisions

---

## Related Resources

### Documentation
- [Official Docs](https://www.tradingview.com/pine-script-docs/)
- [Function Reference](pine-script-reference/)
- [Concepts](pine-script-docs/)

### Tools
- [Decision Trees](decision-trees/) — Make optimal choices
- [Anti-patterns](anti-patterns/) — Avoid common mistakes
- [Knowledge Graph](knowledge-graph.json) — Understand concept relationships

### Learning Resources
- [Examples](examples/) — Practical examples
- [Best Practices](common-mistakes-summary.md): Coding conventions
- [FAQ](常见问题.md): FAQ

---

## Search Tips

### Search by feature
- **Plotting**: plot, plotshape, label, line
- **Strategy**: strategy.entry, strategy.exit, strategy.position_size
- **Data**: array, map, matrix, request.security
- **Time**: time, timenow, barstate.isconfirmed
- **Types**: int, float, string, bool, color

### Search by issue
- **Repainting**: repainting, future leak, lookahead
- **Performance**: performance, loop, calculation
- **Errors**: error, na, runtime error
- **Syntax**: syntax, declaration, scope

---

*Last updated: 2025-10-13*
*Version: Pine Script v6*
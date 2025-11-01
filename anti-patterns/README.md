# Pine Script v6 Anti-Patterns Library

## Overview
This library collects common mistake patterns (anti-patterns) in Pine Script development and provides the correct alternatives. Learning these anti-patterns helps you avoid pitfalls and write more reliable code.

## Categories

### 🚫 Core Anti-patterns
1. **[Repainting Errors](./repainting-errors.md)**
   - Future leak
   - Realtime data misuse
   - request.security() mistakes

2. **[Performance Traps](./performance-traps.md)**
   - Overuse of loops
   - Unnecessary recomputation
   - Memory leaks

3. **[Data Structure Misuse](./data-structure-misuse.md)**
   - Array out-of-bounds
   - Type mismatches
   - Inefficient operations

4. **[Logic Errors](./logic-errors.md)**
   - Incorrect conditions
   - Messy state management
   - Timing/ordering issues

### ⚠️ Advanced Anti-patterns
5. **[Strategy Mistakes](./strategy-mistakes.md)**
   - Order management errors
   - Missing risk control
   - Backtest bias

6. **[Alert Mistakes](./alert-mistakes.md)**
   - Wrong alert frequency
   - Message formatting issues
   - Improper conditions

7. **[Visualization Errors](./visualization-errors.md)**
   - Drawing object management
   - Poor color usage
   - Display issues

8. **[Multi-timeframe Errors](./multi-timeframe-errors.md)**
   - Timeframe confusion
   - Data synchronization problems
   - Future leak

## Entry Format

Each anti-pattern includes:

- ❌ Error example: shows the common wrong code
- 🚨 Problem explanation: why it is wrong
- ✅ Correct approach: a proper implementation
- 💡 Best practices: extra tips and guidance

## How to Use

1. Browse the index: find anti-patterns related to your issue
2. Learn by comparison: understand the difference between wrong and right
3. Apply to your code: update your actual scripts
4. Review regularly: avoid repeating mistakes

## Contributing

If you discover a new anti-pattern, contributions are welcome:

1. Create a new file or add to an existing one
2. Follow the entry format
3. Provide a clear problem explanation
4. Include runnable code examples

## Remember

> "Learning from mistakes is one of the best ways to learn. Knowing what not to do is just as important as knowing what to do."

---

**Tip**: This library is updated continuously. Check back regularly to keep your knowledge fresh.
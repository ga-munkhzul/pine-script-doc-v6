# Visualization Selection Decision Tree

## 📊 Starting question: What type of data do I need to display?

```
┌─────────────────────────────────────────────────────┐
│   🎨 Choose the right visualization to communicate your data   │
└─────────────────────────────────────────────────────┘
    │
    └─ 📈 Data type?
        │
        ├─ Continuous data (price, indicator values)
        │   └─ ➡️ **Plot series**
        │
        ├─ Discrete events (signals, markers)
        │   └─ ➡️ **Shapes/Labels**
        │
        ├─ Area ranges (support/resistance, channels)
        │   └─ ➡️ **Lines/Fills/Boxes**
        │
        ├─ Structured data (tables, info panels)
        │   └─ ➡️ **Tables/Backgrounds**
        │
        └─ Complex combinations (multi-element)
            └─ ➡️ **Hybrid solutions**
```

## 📈 Plot series (continuous data)

```
┌─ Choice: Plot series
│
├─ 🎯 Display purpose?
│   │
│   ├─ Single line
│   │   └─ ✅ **Basic plot()**
│   │       ```pine
│   │       // Simple line
│   │       plot(ta.sma(close, 20), "MA20", color.blue)
│   │       ```
│   │
│   ├─ Multiple lines
│   │   └─ ✅ **Multiple plot()**
│   │       ```pine
│   │       // MA combo
│   │       plot(ta.sma(close, 20), "MA20", color.blue)
│   │       plot(ta.sma(close, 50), "MA50", color.red)
│   │       plot(ta.sma(close, 200), "MA200", color.green)
│   │       ```
│   │
│   ├─ Filled area
│   │   └─ ✅ **plot.style_area/histogram**
│   │       ```pine
│   │       // Area chart
│   │       plot(volume, "Volume",
│   │            style=plot.style_histogram,
│   │            color=volume > ta.sma(volume, 20) ? color.green : color.red)
│   │       ```
│   │
│   └─ Special styles
│       └─ ✅ **Various plot styles**
│           ```pine
│           // Columns
│           plot(rsi - 50, style=plot.style_columns)
│
│           // Step line
│           plot(value, style=plot.style_stepline)
│
│           // Circle markers
│           plot(value, style=plot.style_circles)
│           ```
│
├─ 🎨 Style customization
│   │
│   ├─ Color changes
│   │   ```pine
│   │   // Dynamic color
│   │   maColor = ma20 > ma50 ? color.green : color.red
│   │   plot(ma20, color=maColor)
│   │   ```
│   │
│   ├─ Line style
│   │   ```pine
│   │   plot(ma20,
│   │       linewidth=2,        // Line width
│   │       style=plot.style_line,
│   │       trackprice=true)     // Horizontal line
│   │   ```
│   │
│   └─ Display scope
│       ```pine
│       plot(ma20,
│           display=display.pane,     // Display pane
│           join=true)               // Connect na values
│       ```
│
└─ 📊 Special applications
    ├─ Multiple data series
    │   ```pine
    │   // Use plotchar to display multiple values
    │   plotchar(series1, "Series 1", "A", location.top)
    │   plotchar(series2, "Series 2", "B", location.bottom)
    │   ```
    │
    ├─ Background highlight
    │   ```pine
    │   // Background coloring
    │   bgcolor(rsi > 70 ? color.new(color.red, 90) : na)
    │   ```
    │
    └─ Price markers
        ```pine
        // Mark on price
        plotshape(signal, style=shape.triangleup,
                 location=location.belowbar)
        ```
```

## 📍 Shapes/Labels (discrete events)

```
┌─ Choice: Shapes/Labels
│
├─ 🎯 Event type?
│   │
│   ├─ Simple markers
│   │   └─ ✅ **plotshape()**
│   │       ```pine
│   │       // Buy signal
│   │       plotshape(buySignal,
│   │           title="Buy",
│   │           style=shape.labelup,
│   │           location=location.belowbar,
│   │           color=color.green,
│   │           text="BUY")
│   │       ```
│   │
│   ├─ Custom labels
│   │   └─ ✅ **label.new()**
│   │       ```pine
│   │       // Detailed info label
│   │       if buySignal
│   │           label.new(bar_index, low,
│   │               text="Buy\nPrice: " + str.tostring(close),
│   │               color=color.green,
│   │               style=label.style_label_up,
│   │               size=size.small)
│   │       ```
│   │
│   └─ Multiple shapes
│       └─ ✅ **Different shape.style**
│           ```pine
│           // Arrow
│           plotshape(upSignal, style=shape.triangleup)
│
│           // Circle
│           plotshape(level, style=shape.circle)
│
│           // Square
│           plotshape(signal, style=shape.square)
│
│           // X mark
│           plotshape(error, style=shape.xcross)
│           ```
│
├─ 💡 Position control
│   │
│   ├─ Relative positions
│   │   ```pine
│   │   // Position relative to bar
│   │   location.abovebar    // Above bar
│   │   location.belowbar    // Below bar
│   │   location.absolute    // Absolute position
│   │   location.top         // Top
│   │   location.bottom      // Bottom
│   │   ```
│   │
│   ├─ Absolute position
│   │   ```pine
│   │   // Specify exact price level
│   │   label.new(bar_index, 100.0, "Horizontal line")
│   │   ```
│   │
│   └─ Offset position
│       ```pine
│       // Offset position
│       label.new(bar_index + 2, high, "Future marker")
│       ```
│
├─ 🎨 Style options
│   │
│   ├─ Color and size
│   │   ```pine
│   │   plotshape(signal,
│   │       color=color.blue,
│   │       size=size.normal,     // small/normal/large/huge
│   │       offset=0)             // Offset
│   │   ```
│   │
│   ├─ Text formatting
│   │   ```pine
│   │   label.new(...,
│   │       textcolor=color.white,
│   │       fontfamily=font.family.default,
│   │       size=size.normal)
│   │   ```
│   │
│   └─ Display control
│       ```pine
│       // Conditional display
│       if showLabels and condition
│           label.new(...)
│
│       // Limit number
│       var int labelCount = 0
│       if labelCount < 10
│           labelCount += 1
│           label.new(...)
│       ```
│
└─ 🔄 Management strategies
    ├─ Clean old labels
    │   ```pine
    │   // Save label references
    │   var label[] labelArray = array.new<label>()
    │
    │   // Add new labels
    │   if condition
    │       array.push(labelArray, label.new(...))
    │
    │   // Clean old labels
    │   if array.size(labelArray) > 20
    │       label.delete(array.shift(labelArray))
    │   ```
    │
    └─ Update labels
        ```pine
        // Update existing labels
        var label myLabel = na
        if na(myLabel)
            myLabel := label.new(...)
        else
            label.set_text(myLabel, newText)
            label.set_color(myLabel, newColor)
        ```
```

## 📏 Lines/Fills/Boxes (area ranges)

```
┌─ Choice: Lines/Fills/Boxes
│
├─ 📐 What do you need to draw?
│   │
│   ├─ Horizontal/vertical lines
│   │   └─ ✅ **hline/vline**
│   │       ```pine
│   │       // Horizontal line
│   │       hline(100, "Resistance", color.red, linestyle=hline.style_dashed)
│   │
│   │       // Vertical line
│   │       vline(bar_index, "Time marker", color.blue)
│   │       ```
│   │
│   ├─ Trend line
│   │   └─ ✅ **line.new()**
│   │       ```pine
│   │       // Draw trend line
│   │       if trendStart
│   │           line.new(x1=bar_index[1], y1=high[1],
│   │                    x2=bar_index, y2=high,
│   │                    color=color.blue,
│   │                    width=2,
│   │                    style=line.style_solid,
│   │                    extend=extend.right)
│   │       ```
│   │
│   ├─ Filled area
│   │   └─ ✅ **fill()**
│   │       ```pine
│   │       // Fill between two lines
│   │       p1 = plot(ta.sma(close, 20))
│   │       p2 = plot(ta.sma(close, 50))
│   │       fill(p1, p2, color=color.new(color.blue, 90))
│   │       ```
│   │
│   └─ Rectangle area
│       └─ ✅ **box.new()**
│           ```pine
│           // Draw rectangle
│           if zoneStart
│               box.new(left=bar_index[10], top=high[10],
│                       right=bar_index, bottom=low,
│                       border_color=color.green,
│                       bgcolor=color.new(color.green, 90))
│           ```
│
├─ 🎨 Style customization
│   │
│   ├─ Line styles
│   │   ```pine
│   │   line.style_solid      // Solid line
│   │   line.style_dashed     // Dashed line
│   │   line.style_dotted     // Dotted line
│   │   line.style_arrow_right  // Right arrow
│   │   ```
│   │
│   ├─ Extension options
│   │   ```pine
│   │   extend.none          // No extension
│   │   extend.right          // Extend right
│   │   extend.left          // Extend left
│   │   extend.both          // Extend both
│   │   ```
│   │
│   └─ Fill transparency
│       ```pine
│       // Control transparency
│       fill(p1, p2, color=color.new(color.blue, 80))  // 80% transparency
│       ```
│
├─ 🔧 Dynamic updates
│   │
│   ├─ Move lines
│   │   ```pine
│   │   // Save line reference and update
│   │   var line trendLine = na
│   │   if na(trendLine)
│   │       trendLine := line.new(...)
│   │   else
│   │       line.set_x2(trendLine, bar_index)
│   │       line.set_y2(trendLine, currentPrice)
│   │   ```
│   │
│   ├─ Delete objects
│   │   ```pine
│   │   // Clean up lines
│   │   line.delete(oldLine)
│   │   box.delete(oldBox)
│   │   ```
│   │
│   └─ Batch management
│       ```pine
│       // Batch manage drawing objects
│       var line[] lines = array.new<line>()
│       // Add, update, delete logic
│       ```
│
└─ 📊 Application scenarios
    ├─ Support and resistance
    │   ```pine
    │   // Draw support and resistance
    │   hline(support, "Support", color.green)
    │   hline(resistance, "Resistance", color.red)
    │   fill(support, resistance, color=color.gray)
    │   ```
    │
    ├─ Channels
    │   ```pine
    │   // Price channel
    │   upper = ma + 2 * ta.stdev(close, 20)
    │   lower = ma - 2 * ta.stdev(close, 20)
    │   p1 = plot(upper)
    │   p2 = plot(lower)
    │   fill(p1, p2, color=color.new(color.blue, 90))
    │   ```
    │
    └─ Time window
        ```pine
        // Mark time range
        box.new(startBar, topPrice, endBar, bottomPrice,
                bgcolor=color.new(color.yellow, 90))
        ```
```

## 📋 Tables/Backgrounds (structured data)

```
┌─ Choice: Tables/Backgrounds
│
├─ 📊 Info panel type?
│   │
│   ├─ Simple info display
│   │   └─ ✅ **Text label combination**
│   │       ```pine
│   │       // Create info panel with labels
│   │       var label infoPanel = na
│   │       if na(infoPanel)
│   │       infoPanel := label.new(
│   │           x=bar_index, y=highest,
│   │           text="RSI: " + str.tostring(rsi),
│   │           style=label.style_label_down,
│   │           size=size.normal)
│   │       else
│   │       label.set_text(infoPanel, updateText)
│   │       ```
│   │
│   ├─ Tabular data
│   │   └─ ✅ **table.new()**
│   │       ```pine
│   │       // Create table
│   │       var table infoTable = table.new(position.top_right, 2, 4,
│   │           bgcolor=color.white,
│   │           border_width=1)
│   │
│   │       // Set table content
│   │       if barstate.islast
│   │           table.cell(infoTable, 0, 0, "Indicator", bgcolor=color.gray)
│   │           table.cell(infoTable, 1, 0, "Value", bgcolor=color.gray)
│   │           table.cell(infoTable, 0, 1, "RSI")
│   │           table.cell(infoTable, 1, 1, str.tostring(rsi, "#.##"))
│   │           table.cell(infoTable, 0, 2, "MACD")
│   │           table.cell(infoTable, 1, 2, str.tostring(macd, "#.####"))
│   │       ```
│   │
│   └─ Background highlight
│       └─ ✅ **bgcolor()**
│           ```pine
│           // Conditional background coloring
│           bgColor = rsi > 70 ? color.new(color.red, 90) :
│                      rsi < 30 ? color.new(color.green, 90) :
│                      na
│           bgcolor(bgColor)
│           ```
│
├─ 🎨 Table styles
│   │
│   ├─ Position settings
│   │   ```pine
│   │   position.top_left      // Top-left
│   │   position.top_center    // Top-center
│   │   position.top_right     // Top-right
│   │   position.middle_left   // Middle-left
│   │   position.middle_center // Center
│   │   position.middle_right  // Middle-right
│   │   position.bottom_left   // Bottom-left
│   │   position.bottom_center // Bottom-center
│   │   position.bottom_right  // Bottom-right
│   │   ```
│   │
│   ├─ Cell styles
│   │   ```pine
│   │   table.cell(table_id, col, row, text,
│   │       bgcolor=color.blue,        // Background color
│   │       text_color=color.white,    // Text color
│   │       text_size=size.normal,     // Text size
│   │       text_halign=text.align_left,   // Horizontal alignment
│   │       text_valign=text.align_top)    // Vertical alignment
│   │   ```
│   │
│   └─ Border settings
│       ```pine
│       table.new(...,
│           border_width=1,          // Border width
│           border_color=color.gray) // 边框颜色
│       ```
│
├─ 💡 Best practices
│   │
│   ├─ Update strategy
│   │   ```pine
│   │   // Update only on last bar
│   │   if barstate.islast
│   │       updateTable()
│   │   ```
│   │
│   ├─ Performance optimization
│   │   ```pine
│   │   // Avoid creating table on every bar
│   │   var table myTable = na
│   │   if na(myTable)
│   │       myTable := table.new(...)
│   │   ```
│   │
│   └─ Responsive design
│       ```pine
│       // Adjust based on screen size
│       tableSize = syminfo.isbinary ? 2 : 4
│       ```
│
└─ 📋 Advanced features
    ├─ Merge cells
    │   ```pine
    │   // Pine Script v6 supports merging cells
    │   table.merge_cells(table, 0, 0, 1, 0)
    │   ```
    │
    ├─ Gradient background
    │   ```pine
    │   // Create gradient effect
    │   for i = 0 to 10
    │       alpha = i * 10
    │       bgcolor(color.new(color.blue, alpha))
    │   ```
    │
    └─ Dynamic content
        ```pine
        // Show different content based on conditions
        table.cell(table, 0, 0,
            marketStatus ? "Market open" : "Market closed",
            marketStatus ? color.green : color.red)
        ```
```

## 🎨 Hybrid solutions

```
┌─ Choice: Mix multiple visualization methods
│
├─ 📊 Composite chart example
│   └─ ✅ **Complete technical analysis layout**
│       ```pine
│       // 1. Background
│       bgcolor(ma20 > ma50 ? color.new(color.green, 95) :
│                          color.new(color.red, 95))
│
│       // 2. Main chart - Price and moving averages
│       plot(close, "Close", color.black)
│       plot(ma20, "MA20", color.blue)
│       plot(ma50, "MA50", color.red)
│
│       // 3. Signal markers
│       plotshape(buySignal, "Buy", shape.triangleup,
│                  location.belowbar, color.green, size=size.small)
│       plotshape(sellSignal, "Sell", shape.triangledown,
│                  location.abovebar, color.red, size=size.small)
│
│       // 4. Support/Resistance
│       hline(support, "Support", color.green, linestyle=hline.style_dashed)
│       hline(resistance, "Resistance", color.red, linestyle=hline.style_dashed)
│
│       // 5. Info panel
│       if barstate.islast
│           createInfoPanel()
│       ```
│
├─ 🎯 Selection principles
│   │
│   ├─ Don't overdecorate
│   │   ❌ Too many colors, markers, lines
│   │   ✅ Keep it simple and clear
│   │
│   ├─ Clear information hierarchy
│   │   - Primary data: most prominent
│   │   - Secondary data: lighter
│   │   - Reference info: minimized
│   │
│   ├─ Color usage conventions
│   │   - Green: bullish/positive
│   │   - Red: bearish/negative
│   │   - Blue: neutral info
│   │   - Yellow/Orange: warning
│   │
│   └─ Performance considerations
│       - Limit number of drawing objects
│       - Reasonable update frequency
│       - Avoid redundant drawing
│
└─ 📝 Debugging tips
    ├─ Use display parameter to control visibility
    │   ```pine
    │   plot(debugValue, display=display.none)  // Data window only
    │   ```
    │
    ├─ Conditionally show debug info
    │   ```pine
    │   if debugMode
    │       label.new(bar_index, high, "Debug: " + str.tostring(value))
    │   ```
    │
    └─ Use different colors to distinguish versions
        ```pine
        v1Color = version == 1 ? color.blue : color.gray
        plot(value1, color=v1Color)
        ```
```

## 📊 Visualization decision checklists

### Plot selection checklist
- [ ] Is the data continuous?
- [ ] Need to show trend?
- [ ] Need to fill area?
- [ ] Need special styles?

### Shape/Label selection checklist
- [ ] Are events discrete?
- [ ] Need to show detailed info?
- [ ] Is marker position important?
- [ ] Need to limit count?

### Line/Box selection checklist
- [ ] Need to connect multiple points?
- [ ] Need to show area?
- [ ] Need to extend lines?
- [ ] Need dynamic updates?

### Table selection checklist
- [ ] Need to display structured data?
- [ ] Need multiple rows and columns?
- [ ] Need real-time updates?
- [ ] Need special styles?

## ⚠️ Common errors to avoid

1. Overplotting: too many elements cause clutter
2. Color conflicts: similar colors are hard to distinguish
3. Performance issues: too many dynamic objects
4. Information overload: showing too much information
5. Overlapping positions: elements block each other
6. Too frequent updates: updating on every tick

Remember: A good visualization clearly conveys information rather than showing everything. Choose the simplest, most effective way to express your data.
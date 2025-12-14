# Coverage Rules of Thumb

## Summary
Formulas for calculating the volume of material needed for different shapes (Rectangles, Triangles, Circles). The goal is to find Cubic Yards.

## Key Details
- **Rectangles / Squares**:
    - Formula: `Length (ft) x Width (ft) x Depth (ft) ÷ 27 = Cubic Yards`
    - Alternative: `Length x Width x Thickness (in inches ÷ 12) = Cubic Feet ÷ 27 = Cubic Yards`
- **Triangles**:
    - Formula: `Base (ft) x Height (ft) ÷ 2 x Depth (ft) ÷ 27 = Cubic Yards`
- **Circles (Holes/Columns)**:
    - Formula: `Pi (3.142) x Radius² x Depth` (all in same unit, e.g., inches or feet), then convert to Cubic Yards.
    - Example: 5ft diameter (2.5ft radius), 2.5ft deep. `3.142 x 2.5² x 2.5 = 49.1 cubic feet`. `49.1 ÷ 27 = 1.82 Cubic Yards`.

## When This Applies
When a customer gives dimensions of their project area.

## What The Customer Usually Asks
- "I have a circle driveway."
- "My area is a weird shape." (Break it down into rectangles and triangles).

## How The Agent Should Respond
"We can break that down. Let's calculate the main rectangle first, then add the other sections. What's the length and width of the main part?"

## Red Flags / Misconceptions
- "Step off area" method: Each step count multiplied by 3 gives roughly feet, but measuring tape is better.
- All results are estimates.

## Internal Notes (Not customer-facing)
- "Proper Planning Prevents Poor Performance."
- Always round up slightly to ensure enough material (see rounding logic).

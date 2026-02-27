# Canvas — Animated Checkerboard

[← Back to Class 07](class07-Feb27.md)

A checkerboard pattern that shifts over time using `oscillateInt()`. Nested `for` loops draw the grid; a time-driven offset controls which squares are lit.

---

## Complete Sketch

```cpp
// ============================================================
// Animated Checkerboard
// ============================================================
// What this does:
//   Draws a checkerboard pattern across the 12×8 matrix.
//   The pattern shifts by one cell on a timer, creating
//   a blinking/crawling effect.
//
// Key concepts:
//   - Nested for loops to cover the full grid
//   - Modulo (%) to alternate on/off
//   - oscillateInt() to animate the offset over time
// ============================================================

#include "TinyFilmFestival.h"

TinyScreen screen;

// --- Animation timing ---
int SHIFT_PERIOD = 2000;  // ms for one full shift cycle

void setup()
{
    screen.begin();
}

void loop()
{
    // The offset shifts between 0 and 1 over time
    // This flips which squares are "on" vs "off"
    int offset = oscillateInt(0, 1, SHIFT_PERIOD);

    screen.beginDraw();
    screen.background(OFF);
    screen.stroke(ON);

    // Walk through every cell on the 12×8 grid
    for (int x = 0; x < 12; x++)
    {
        for (int y = 0; y < 8; y++)
        {
            // (x + y + offset) % 2 alternates the pattern
            // When offset changes, the whole board flips
            if ((x + y + offset) % 2 == 0)
            {
                screen.point(x, y);
            }
        }
    }

    screen.endDraw();
}
```

---

## What to Notice

- `(x + y + offset) % 2` is the core trick — adding `offset` shifts which cells are even vs odd, flipping the pattern.
- `SHIFT_PERIOD` controls how fast the pattern alternates. Smaller = faster flicker.
- The `for` loops cover every pixel. You could change the step size (`x += 2`) or the condition to create different grid densities.

### Tuning

| What to change | Where | Effect |
|---|---|---|
| **Speed** | `SHIFT_PERIOD` | Smaller = faster flicker, larger = slower pulse |
| **Density** | Loop step size (`x += 2`, `y += 2`) | Skip pixels to create a sparser grid |
| **Pattern** | Modulo divisor (`% 3` instead of `% 2`) | Creates stripes or other repeating patterns |
| **Offset range** | `oscillateInt(0, 3, ...)` | Cycles through more pattern variants |

---

## Going Further

> **Hint:** What if `offset` came from a sensor instead of `oscillateInt()`? What if a sensor controlled the grid spacing?

- Replace `oscillateInt()` with `map(sensorValue, ...)` to let a sensor shift the pattern.
- Use `map()` to control the step size in the `for` loops — pressure could control grid density.
- Add a second `oscillateInt()` to shift rows and columns independently.

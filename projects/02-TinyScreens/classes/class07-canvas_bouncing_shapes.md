# Canvas — Bouncing Shapes with Phase Offsets

[← Back to Class 07](class07-Feb27.md)

Three shapes — a circle, a rectangle, and a dot — each animated independently with `oscillateInt()` using different periods and phase offsets. They move at different speeds creating a layered, organic composition.

---

## Complete Sketch

```cpp
// ============================================================
// Bouncing Shapes with Phase Offsets
// ============================================================
// What this does:
//   Three shapes move independently across the screen.
//   Each uses oscillateInt() with a different period and
//   phase offset so they never sync up exactly.
//
// Key concepts:
//   - Multiple oscillateInt() calls with different timing
//   - Phase offsets (0.0–1.0) to stagger motion
//   - Layered canvas drawing (order matters)
// ============================================================

#include "TinyFilmFestival.h"

TinyScreen screen;

// --- Shape timing ---
// Each shape has its own period and phase offset
// so they move at different rates and start at different points
int CIRCLE_PERIOD = 1500;      // ms — circle bounce cycle
int RECT_PERIOD = 2200;        // ms — rectangle bounce cycle
int DOT_PERIOD = 800;          // ms — dot bounce cycle

float CIRCLE_PHASE = 0.0;     // starts at the beginning
float RECT_PHASE = 0.33;      // starts 1/3 through its cycle
float DOT_PHASE = 0.66;       // starts 2/3 through its cycle

void setup()
{
    screen.begin();
}

void loop()
{
    // Each shape gets its own x position from oscillateInt
    int circleX = oscillateInt(1, 10, CIRCLE_PERIOD, CIRCLE_PHASE);
    int rectX = oscillateInt(0, 8, RECT_PERIOD, RECT_PHASE);
    int dotY = oscillateInt(0, 7, DOT_PERIOD, DOT_PHASE);

    // The circle also bounces vertically at a different rate
    int circleY = oscillateInt(1, 6, 1800, 0.5);

    screen.beginDraw();
    screen.background(OFF);
    screen.stroke(ON);

    // Rectangle — outline only, slides horizontally
    screen.noFill();
    screen.rect(rectX, 5, 4, 3);

    // Circle — filled, moves in both x and y
    screen.fill(ON);
    screen.circle(circleX, circleY, 3);

    // Dot — just a point, bounces vertically at the right edge
    screen.point(11, dotY);

    screen.endDraw();
}
```

---

## What to Notice

- Phase offsets (`0.33`, `0.66`) mean shapes start at different points in their cycle even if they share the same period. This prevents everything from moving in lockstep.
- The circle uses **two** `oscillateInt()` calls — one for x, one for y — creating diagonal/curved motion paths.
- Drawing order matters: shapes drawn later appear "on top" if they overlap.

### Tuning

| What to change | Where | Effect |
|---|---|---|
| **Speed per shape** | `CIRCLE_PERIOD`, `RECT_PERIOD`, `DOT_PERIOD` | Change how fast each shape moves independently |
| **Phase stagger** | `CIRCLE_PHASE`, `RECT_PHASE`, `DOT_PHASE` | Change where each shape starts in its cycle (0.0–1.0) |
| **Movement range** | `oscillateInt()` min/max values | Control how far each shape travels |
| **Shape types** | Replace `screen.circle()`, `screen.rect()`, `screen.point()` | Use different drawing primitives |
| **Number of shapes** | Add more `oscillateInt()` calls + draw commands | Stack more independent elements |

---

## Going Further

> **Hint:** Each `oscillateInt()` could be replaced by a `map()` call from a sensor. Which shapes would feel most interesting to control directly?

- Replace one shape's `oscillateInt()` with sensor input — keep the others time-driven for a mix of automatic and interactive motion.
- Map a sensor to the period of one shape — pressure controls how fast the circle bounces while the others stay constant.
- Add more shapes with different phase offsets for a richer composition.

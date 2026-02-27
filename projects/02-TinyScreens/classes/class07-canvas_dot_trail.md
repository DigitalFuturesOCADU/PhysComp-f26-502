# Canvas — Dot Trail with History Array

[← Back to Class 07](class07-Feb27.md)

A dot moves across the screen, leaving a trail behind it. The trail uses a circular buffer (array) to remember the last N positions — the same data structure used in [rolling average smoothing](class07-pressure_smoothing.md), applied here as a visual tool.

---

## Complete Sketch

```cpp
// ============================================================
// Dot Trail with History Array
// ============================================================
// What this does:
//   A dot bounces horizontally. An array stores its last N
//   positions, and all stored positions are drawn each frame.
//   Older positions appear as smaller dots, creating a trail.
//
// Key concepts:
//   - Circular buffer (array + index that wraps around)
//   - for loop to draw historical positions
//   - Same buffer pattern used in rolling average smoothing
// ============================================================

#include "TinyFilmFestival.h"

TinyScreen screen;

// --- Trail settings ---
const int TRAIL_LENGTH = 6;           // how many past positions to remember
int trailX[TRAIL_LENGTH];             // x positions
int trailY[TRAIL_LENGTH];             // y positions
int trailIndex = 0;                   // where to write next (circular)

// --- Motion timing ---
int BOUNCE_PERIOD_X = 2000;           // ms — horizontal bounce
int BOUNCE_PERIOD_Y = 1400;           // ms — vertical bounce

// --- Trail update timing ---
int TRAIL_INTERVAL = 80;             // ms between trail snapshots
unsigned long lastTrailUpdate = 0;

void setup()
{
    screen.begin();

    // Initialize trail positions to center
    for (int i = 0; i < TRAIL_LENGTH; i++)
    {
        trailX[i] = 6;
        trailY[i] = 4;
    }
}

void loop()
{
    // Current dot position — driven by time
    int currentX = oscillateInt(0, 11, BOUNCE_PERIOD_X);
    int currentY = oscillateInt(1, 6, BOUNCE_PERIOD_Y, 0.25);

    // Save position to trail at regular intervals
    if (millis() - lastTrailUpdate >= TRAIL_INTERVAL)
    {
        lastTrailUpdate = millis();

        trailX[trailIndex] = currentX;
        trailY[trailIndex] = currentY;
        trailIndex = (trailIndex + 1) % TRAIL_LENGTH;  // wrap around
    }

    screen.beginDraw();
    screen.background(OFF);
    screen.stroke(ON);

    // Draw trail — older positions first (they get drawn under newer ones)
    for (int i = 0; i < TRAIL_LENGTH; i++)
    {
        // Read from oldest to newest using circular index
        int readIndex = (trailIndex + i) % TRAIL_LENGTH;
        screen.point(trailX[readIndex], trailY[readIndex]);
    }

    // Draw current dot as a small circle (stands out from trail)
    screen.fill(ON);
    screen.circle(currentX, currentY, 2);

    screen.endDraw();
}
```

---

## What to Notice

- The circular buffer pattern (`trailIndex = (trailIndex + 1) % TRAIL_LENGTH`) is the same technique used in [rolling average smoothing](class07-pressure_smoothing.md). Learning it here gives you two uses for one concept.
- `TRAIL_LENGTH` controls how many past positions are visible. Longer = more trail, but uses more memory.
- `TRAIL_INTERVAL` controls how far apart the trail dots are spaced in time. Shorter = denser trail.

### Tuning

| What to change | Where | Effect |
|---|---|---|
| **Trail length** | `TRAIL_LENGTH` | More past positions = longer visible trail |
| **Trail density** | `TRAIL_INTERVAL` | Shorter interval = dots closer together |
| **Motion speed** | `BOUNCE_PERIOD_X`, `BOUNCE_PERIOD_Y` | Change how fast the lead dot moves |
| **Motion path** | `oscillateInt()` ranges and phases | Change the shape of the path |
| **Trail appearance** | Drawing commands in the trail `for` loop | Use `screen.circle()` instead of `screen.point()` for larger trail dots |

---

## Going Further

> **Hint:** What if the dot's position came from a distance sensor instead of `oscillateInt()`? The trail would show the *history* of your hand movement.

- Replace `oscillateInt()` with `map(sensorValue, ...)` — the trail becomes a visualization of sensor input over time.
- Use different `TRAIL_INTERVAL` values to show more or less history.
- Draw the trail with decreasing circle sizes (newest = biggest, oldest = smallest) for a comet effect.

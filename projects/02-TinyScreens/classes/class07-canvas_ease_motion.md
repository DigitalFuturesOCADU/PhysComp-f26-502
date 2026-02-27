# Canvas — Smooth Motion with Ease

[← Back to Class 07](class07-Feb27.md)

The previous canvas examples use `oscillateInt()` for motion — a sine wave that cycles endlessly between two values. The `Ease` class works differently: it moves **to a target value once and stops**, with smooth acceleration and deceleration. This makes it ideal for state-driven motion — move somewhere, arrive, decide where to go next.

For full API details, see the [Ease class documentation](https://digitalfuturesocadu.github.io/TinyFilmFestival/docs/#canvas-mode) and the [EaseDemo example](https://digitalfuturesocadu.github.io/TinyFilmFestival/docs/#example-ease-demo).

---

## The Concept

| | `oscillateInt()` | `Ease` |
|---|---|---|
| **Motion** | Cycles back and forth forever | Moves to a target, then stops |
| **Control** | Set min, max, period | Set target and duration |
| **When to use** | Continuous ambient motion | Intentional moves between positions |
| **Sensor potential** | Replace min/max/period | Replace `.to(target, ...)` with sensor value |

`Ease` objects remember their current value and interpolate toward the target over the specified duration. You check `.done()` to know when the motion is complete, then decide what to do next.

---

## Complete Sketch

A circle visits each corner of the matrix and the center, easing smoothly between positions. When it arrives at a target, it pauses briefly then moves to the next one. The circle's diameter also eases — it shrinks while moving and grows when stopped.

```cpp
// ============================================================
// Smooth Motion with Ease
// ============================================================
// What this does:
//   A circle moves between 5 positions (4 corners + center)
//   using the Ease class for smooth acceleration/deceleration.
//   The circle shrinks while in motion and grows when stopped.
//
// Key concepts:
//   - Ease objects for position (x, y) and size (diameter)
//   - .to(target, duration) to start a smooth move
//   - .done() to detect arrival and trigger the next move
//   - .intValue() to get pixel-ready coordinates
// ============================================================

#include "TinyFilmFestival.h"

TinyScreen screen;

// --- Ease objects ---
// Each one smoothly interpolates from its current value to a target
Ease xPos(0);       // horizontal position — starts at left
Ease yPos(0);       // vertical position — starts at top
Ease diameter(6);   // circle size — starts medium

// --- Target positions ---
// The circle visits these positions in order, then loops
// Each row is {x, y} — corners and center of the 12×8 matrix
int targets[][2] = {
    {2, 2},      // top-left area
    {9, 2},      // top-right area
    {9, 5},      // bottom-right area
    {2, 5},      // bottom-left area
    {6, 4}       // center
};
int NUM_TARGETS = 5;
int currentTarget = 0;

// --- Timing ---
int MOVE_DURATION = 800;     // ms — how long each move takes
int PAUSE_DURATION = 400;    // ms — how long to wait at each stop
unsigned long arrivedTime = 0;
bool waiting = false;

void setup()
{
    screen.begin();

    // Start the first move
    xPos.to(targets[0][0], MOVE_DURATION);
    yPos.to(targets[0][1], MOVE_DURATION);
    diameter.to(3, 200);   // shrink while moving
}

void loop()
{
    // --- State logic ---
    if (xPos.done() && yPos.done())
    {
        if (!waiting)
        {
            // Just arrived — start the pause
            waiting = true;
            arrivedTime = millis();
            diameter.to(6, 200);   // grow when stopped
        }

        if (waiting && millis() - arrivedTime >= PAUSE_DURATION)
        {
            // Pause is over — move to next target
            waiting = false;
            currentTarget = (currentTarget + 1) % NUM_TARGETS;

            xPos.to(targets[currentTarget][0], MOVE_DURATION);
            yPos.to(targets[currentTarget][1], MOVE_DURATION);
            diameter.to(3, 200);   // shrink while moving
        }
    }

    // --- Draw ---
    screen.beginDraw();
    screen.background(OFF);
    screen.stroke(ON);
    screen.fill(ON);
    screen.circle(xPos.intValue(), yPos.intValue(), diameter.intValue());
    screen.endDraw();
}
```

---

## What to Notice

- **`Ease` objects work like animated variables.** You set a target with `.to()` and the value smoothly moves there on its own. Calling `.intValue()` each frame gives you the current position — no manual math needed.
- **`.done()` is your trigger.** It returns `true` only when the object has reached its target. This lets you sequence motions: arrive → pause → pick next target → move again.
- **Multiple Ease objects are independent.** `xPos`, `yPos`, and `diameter` each animate at their own pace. The diameter finishes its short 200ms ease before the position finishes its 800ms move.
- **The `waiting` flag is a simple state machine.** It distinguishes "just arrived" (start the pause timer + grow the circle) from "still waiting" (check if the pause is over).
- **Unlike `oscillateInt()`, Ease gives you control over *when* motion happens.** You decide the destination and duration at runtime. This makes it a natural fit for sensor-triggered motion — "when a threshold is crossed, ease to a new position."

### Tuning

| What to change | Where | Effect |
|---|---|---|
| **Move speed** | `MOVE_DURATION` | Shorter = snappier motion; longer = slower, more graceful |
| **Pause length** | `PAUSE_DURATION` | Shorter = restless; longer = contemplative |
| **Target positions** | `targets[][]` array | Change the path — try a zigzag, a spiral, or random positions |
| **Number of targets** | Add/remove rows in `targets[][]` + update `NUM_TARGETS` | More stops = longer sequence |
| **Size change** | `diameter.to(...)` values | Bigger range = more dramatic breathing effect |
| **Shape** | Replace `screen.circle(...)` with `screen.rect(...)` | Different geometry, same motion logic |

---

## Going Further

> **Hint:** Replace the automatic target sequence with sensor-driven targets. Use `map(sensorValue, ...)` to set where the circle eases to, instead of cycling through a fixed array.

- Use a distance sensor to set the x target: `xPos.to(map(distance, 2, 100, 0, 11), 300)` — the circle smoothly follows your hand, but with eased motion instead of jittery direct mapping.
- Use a pressure threshold to trigger the next target: replace the automatic pause timer with `if (pressureZone == 2) { /* move to next target */ }`.
- Combine Ease with `oscillateInt()`: ease the position while oscillating the diameter for a shape that breathes continuously but moves deliberately.

# Animation — Moving Animation with setPosition

[← Back to Class 07](class07-Feb27.md)

An animation plays while `oscillateInt()` moves it across the screen like a sprite. The animation's visual content stays the same, but its position on the matrix changes over time.

This example requires an animation `.h` file exported from the [LED Matrix Editor](https://ledmatrix-editor.arduino.cc/). If you need a refresher on creating and exporting animations, see [Class 05 — Creating an Animation](class05-Feb06.md#creating-an-animation).

In the code below, `myAnimA` is a placeholder name — replace it with the actual variable name from your exported `.h` file.

---

## Complete Sketch

```cpp
// ============================================================
// Moving Animation with setPosition
// ============================================================
// What this does:
//   Plays an animation and slides it horizontally across
//   the screen using setPosition(). The animation itself
//   loops normally — its screen position is what changes.
//
// Key concepts:
//   - setPosition(x, y) offsets the animation on the matrix
//   - Position persists across frames (set it once or update it)
//   - Combining animation playback with dynamic positioning
// ============================================================

#include "TinyFilmFestival.h"
#include "myAnimA.h"    // ← replace with your animation file

TinyScreen screen;

// --- Motion timing ---
int SLIDE_PERIOD = 3000;   // ms — how long one slide cycle takes

void setup()
{
    screen.begin();
    screen.play(myAnimA, LOOP);
}

void loop()
{
    // Slide the animation horizontally
    int xOffset = oscillateInt(-4, 4, SLIDE_PERIOD);
    screen.setPosition(xOffset, 0);

    screen.update();
}
```

---

## What to Notice

- `setPosition(x, y)` offsets the entire animation from its default position. Negative values shift left/up, positive shift right/down.
- The animation continues playing its frames normally — only its location on the matrix moves.
- You can move in both x and y simultaneously for diagonal motion.
- Position persists — once set, it stays until you change it again.

### Tuning

| What to change | Where | Effect |
|---|---|---|
| **Slide range** | `oscillateInt(-4, 4, ...)` min/max | How far the animation travels from center |
| **Slide speed** | `SLIDE_PERIOD` | How fast the animation slides back and forth |
| **Slide axis** | `screen.setPosition(0, yOffset)` | Slide vertically instead of horizontally |
| **Motion path** | Add a second `oscillateInt()` for y | Create diagonal or circular motion |
| **Layer positioning** | `screen.setPositionOnLayer(layer, x, y)` | Move individual layers independently (combine with [Layered Animations](class07-animation_layers.md)) |

---

## Going Further

> **Hint:** Replace `oscillateInt()` with `map(sensorValue, ...)` to steer the animation with your hand or grip pressure.

- Use `map(distanceToHand, 2, 100, -6, 6)` to control x position with the distance sensor — the animation follows your hand.
- Add vertical positioning driven by a second sensor or a second `oscillateInt()`.
- Combine with layering: two layers, each with its own `setPositionOnLayer()`, moving independently.

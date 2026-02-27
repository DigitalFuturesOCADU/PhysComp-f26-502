# Animation — Sequential Animation Playback

[← Back to Class 07](class07-Feb27.md)

Two (or more) animations play one after another in a sequence. When the first finishes, the second starts, then it cycles back. This is useful for telling a visual story or for state-based feedback.

This example requires two animation `.h` files exported from the [LED Matrix Editor](https://ledmatrix-editor.arduino.cc/). If you need a refresher on creating and exporting animations, see [Class 05 — Creating an Animation](class05-Feb06.md#creating-an-animation).

In the code below, `myAnimA` and `myAnimB` are placeholder names — replace them with the actual variable names from your exported `.h` files.

---

## Complete Sketch

```cpp
// ============================================================
// Sequential Animation Playback
// ============================================================
// What this does:
//   Plays Animation A once, then Animation B once, then
//   back to A, in an endless A → B → A → B cycle.
//
// Key concepts:
//   - ONCE play mode — plays exactly one cycle, then stops
//   - isComplete() — returns true when a ONCE animation finishes
//   - State tracking with a variable to know which animation is active
// ============================================================

#include "TinyFilmFestival.h"
#include "myAnimA.h"    // ← replace with your animation files
#include "myAnimB.h"

TinyScreen screen;

// --- State tracking ---
int currentAnim = 0;    // 0 = playing A, 1 = playing B

void setup()
{
    screen.begin();
    screen.play(myAnimA, ONCE);
    currentAnim = 0;
}

void loop()
{
    // When the current animation finishes, switch to the other
    if (screen.isComplete())
    {
        if (currentAnim == 0)
        {
            screen.play(myAnimB, ONCE);
            currentAnim = 1;
        }
        else
        {
            screen.play(myAnimA, ONCE);
            currentAnim = 0;
        }
    }

    screen.update();
}
```

---

## What to Notice

- `ONCE` makes the animation play its frames exactly one time, then stop on the last frame.
- `isComplete()` returns `true` only after a `ONCE` animation has finished all its frames. It never returns `true` for `LOOP` animations.
- The `currentAnim` variable tracks which animation is active so the `if` block knows which one to start next.
- You can extend this to three or more animations by using `0`, `1`, `2`, etc.

### Tuning

| What to change | Where | Effect |
|---|---|---|
| **Number of animations** | Add more `.h` includes + states | Chain 3, 4, or more animations |
| **Speed per animation** | `screen.setSpeed(fps)` before each `play()` | Each animation plays at a different speed |
| **Looping final animation** | Change the last `play()` to `LOOP` | Sequence ends on a looping animation |
| **Pause between animations** | Add a `delay()` or `millis()` timer before the next `play()` | Brief gap between transitions |
| **Reverse order** | Swap `myAnimA` and `myAnimB` in the else block | Change the sequence direction |

---

## Going Further

> **Hint:** Replace the automatic `isComplete()` trigger with a sensor threshold to switch animations manually.

- Use a pressure threshold to trigger the next animation — light press plays A, firm press plays B.
- Use the distance sensor: hand close plays one animation, hand far plays another.
- Combine with [Layered Animations](class07-animation_layers.md) — sequence on one layer while a background loop plays on another.

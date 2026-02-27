# Hybrid Mode — Canvas Overlay on Animation

[← Back to Class 07](class07-Feb27.md)

An animation plays as a background while Canvas drawing commands add a moving overlay on top. The animation runs normally — the overlay composites onto each frame without erasing it.

This sketch uses a ready-made `.h` file from the [TinyFilmFestival example animations](https://github.com/DigitalFuturesOCADU/TinyFilmFestival/tree/main/exampleAnimations). Download it and place it in the same folder as your `.ino` file.

> **Swapping animations:** Change the `#include` line to a different `.h` file and update the variable name in `screen.play()` to match (the variable name is the filename without `.h`). You can use any file from the [example animations folder](https://github.com/DigitalFuturesOCADU/TinyFilmFestival/tree/main/exampleAnimations), or create your own with the [LED Matrix Editor](https://ledmatrix-editor.arduino.cc/). See the [Hybrid Mode guide](https://digitalfuturesocadu.github.io/TinyFilmFestival/docs/#hybrid-mode) for full documentation.

---

## Complete Sketch

```cpp
// ============================================================
// Hybrid Mode — Canvas Overlay on Animation
// ============================================================
// What this does:
//   Plays the landscape animation as a looping background.
//   On top of each frame, draws a bouncing circle and a
//   horizontal line that sweeps up and down — all using
//   Canvas drawing commands inside beginOverlay/endOverlay.
//
// Key concepts:
//   - beginOverlay() / endOverlay() draw ON TOP of animation
//   - All Canvas drawing methods work inside the overlay
//   - oscillateInt() drives the overlay motion independently
//   - The animation keeps playing underneath without being erased
// ============================================================

#include "TinyFilmFestival.h"
#include "landscape.h"         // download from exampleAnimations folder

TinyScreen screen;
Animation bgAnim = landscape;

void setup()
{
    screen.begin();
    screen.play(bgAnim, LOOP);
}

void loop()
{
    // 1. Advance the animation frame
    screen.update();

    // 2. Calculate overlay positions
    int circleX = oscillateInt(2, 9, 3000);        // horizontal bounce
    int circleY = oscillateInt(1, 4, 2000, 0.25);  // vertical bounce, phase-shifted
    int lineY   = oscillateInt(5, 7, 4000);         // slow sweep across bottom rows

    // 3. Draw overlay on top of the current animation frame
    screen.beginOverlay();

    screen.stroke(ON);
    screen.circle(circleX, circleY, 2);             // bouncing circle
    screen.line(0, lineY, 11, lineY);               // sweeping horizontal line

    screen.endOverlay();
}
```

---

## What to Notice

- **`beginOverlay()` vs `beginDraw()`** — `beginDraw()` clears the matrix before drawing (Canvas Mode). `beginOverlay()` preserves the current animation frame so your drawing appears on top.
- **Order matters** — call `screen.update()` first to render the animation frame, *then* `beginOverlay()` to draw over it.
- The overlay is redrawn every loop iteration on top of the latest animation frame. There is no leftover — each frame starts fresh from the animation.
- Lit overlay pixels combine with lit animation pixels. Since each LED is either on or off, overlapping lit pixels simply stay on.

### Tuning

| What to change | Where | Effect |
|---|---|---|
| **Overlay shape** | Inside `beginOverlay()` / `endOverlay()` | Use `point()`, `rect()`, `line()`, `circle()`, `text()` — any Canvas drawing method |
| **Motion speed** | `oscillateInt()` period values | Faster or slower overlay movement independent of the animation |
| **Background animation** | `#include` and `screen.play()` | Swap `landscape.h` for any other `.h` file |
| **Animation speed** | `screen.setSpeed(multiplier)` in `setup()` | Slow down or speed up the background animation |
| **Static overlay** | Remove `oscillateInt()`, use fixed coordinates | Draw a fixed indicator or border that doesn't move |

---

## Going Further

> **Hint:** Replace `oscillateInt()` with `map(sensorValue, ...)` to control the overlay with your hand or grip pressure — the background animation keeps playing no matter what.

- Map distance sensor to the circle's position — your hand steers the overlay while the animation loops beneath.
- Use a pressure threshold to toggle the overlay on/off: draw nothing when not pressed, draw when pressed.
- Combine with `setPosition()` to move the **background animation** with one sensor while the **overlay** responds to a second sensor.
- Use `Ease` instead of `oscillateInt()` for one-shot overlay transitions triggered by sensor events.

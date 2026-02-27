# Animation — Layered Animations

[← Back to Class 07](class07-Feb27.md)

Two animations play simultaneously on separate layers, each at its own speed. Layers composite automatically — lit pixels from any layer appear on the screen.

This example requires animation `.h` files exported from the [LED Matrix Editor](https://ledmatrix-editor.arduino.cc/). If you need a refresher on creating and exporting animations, see [Class 05 — Creating an Animation](class05-Feb06.md#creating-an-animation).

In the code below, `myAnimA` and `myAnimB` are placeholder names — replace them with the actual variable names from your exported `.h` files (e.g., `#include "heartbeat.h"` and use `heartbeat`).

---

## Complete Sketch

```cpp
// ============================================================
// Layered Animations
// ============================================================
// What this does:
//   Plays two animations at the same time on separate layers.
//   Each layer has its own speed, so they drift in and out
//   of sync with each other.
//
// Key concepts:
//   - addLayer() to create a second animation layer
//   - playOnLayer() to assign an animation to a specific layer
//   - setSpeedOnLayer() for independent speed control
// ============================================================

#include "TinyFilmFestival.h"
#include "myAnimA.h"    // ← replace with your animation file
#include "myAnimB.h"    // ← replace with your second animation file

TinyScreen screen;

int secondLayer = -1;  // will hold the layer index after addLayer()

void setup()
{
    screen.begin();

    // Create a second layer (layer 0 is the default)
    secondLayer = screen.addLayer();

    // Play animation A on the default layer (layer 0)
    screen.play(myAnimA, LOOP);
    screen.setSpeed(1.0);               // normal speed

    // Play animation B on the second layer
    if (secondLayer >= 0)
    {
        screen.playOnLayer(secondLayer, myAnimB, LOOP);
        screen.setSpeedOnLayer(secondLayer, 0.5);   // half speed
    }
}

void loop()
{
    screen.update();  // advances both layers
}
```

---

## What to Notice

- `addLayer()` returns a layer index (or -1 if max layers exceeded). Always check before using.
- Each layer plays independently — different animations, different speeds, different play modes.
- Lit pixels from any layer show on screen. Overlapping lit pixels are simply both on (there's no blending — every LED is either on or off).
- Up to 4 additional layers can be created (5 total including the default layer 0).

### Tuning

| What to change | Where | Effect |
|---|---|---|
| **Layer speed** | `screen.setSpeedOnLayer(layer, multiplier)` | Change playback speed per layer independently |
| **Play mode** | `LOOP`, `ONCE`, `BOOMERANG` in `playOnLayer()` | Each layer can use a different play mode |
| **Number of layers** | Add more `addLayer()` calls (up to 4 additional) | Stack more animations simultaneously |
| **Individual layer control** | `pauseLayer()`, `resumeLayer()`, `stopLayer()` | Control each layer independently in `loop()` |

---

## Going Further

> **Hint:** What if one layer's speed was controlled by a sensor while the other stayed constant? Use `setSpeedOnLayer()` inside `loop()` instead of `setup()`.

- Move `setSpeedOnLayer()` into `loop()` and drive it with `map(sensorValue, ...)` — one animation responds to input while the other plays at a fixed pace.
- Use `pauseLayer()` and `resumeLayer()` triggered by thresholds to freeze one layer while the other keeps playing.
- Try `BOOMERANG` mode on one layer and `LOOP` on the other for contrasting motion styles.

# Animation — Layered Animations

[← Back to Class 07](class07-Feb27.md)

Two animations play simultaneously on separate layers, each at its own speed. Layers composite automatically — lit pixels from any layer appear on the screen.

This sketch uses ready-made `.h` files from the [TinyFilmFestival example animations](https://github.com/DigitalFuturesOCADU/TinyFilmFestival/tree/main/exampleAnimations). Download them and place them in the same folder as your `.ino` file.

> **Swapping animations:** Change the `#include` line to a different `.h` file and update the variable name in `screen.play()` to match (the variable name is the filename without `.h`). You can use any file from the [example animations folder](https://github.com/DigitalFuturesOCADU/TinyFilmFestival/tree/main/exampleAnimations), or create your own with the [LED Matrix Editor](https://ledmatrix-editor.arduino.cc/). See the [Animation Mode guide](https://digitalfuturesocadu.github.io/TinyFilmFestival/docs/#animation-mode) for full documentation.

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
#include "landscape.h"   // download from exampleAnimations folder
#include "idle.h"        // download from exampleAnimations folder

TinyScreen screen;

int secondLayer = -1;  // will hold the layer index after addLayer()

void setup()
{
    screen.begin();

    // Create a second layer (layer 0 is the default)
    secondLayer = screen.addLayer();

    // Play landscape on the default layer (layer 0)
    screen.play(landscape, LOOP);
    screen.setSpeed(1.0);               // normal speed

    // Play idle character on the second layer
    if (secondLayer >= 0)
    {
        screen.playOnLayer(secondLayer, idle, LOOP);
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

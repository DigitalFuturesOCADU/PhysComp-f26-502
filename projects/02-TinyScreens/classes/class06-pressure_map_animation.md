# Pressure Sensor → Animation: Step by Step

[← Back to Class 06](class06-Feb13.md)

This walkthrough takes you from a blank animation to a sensor-controlled animation in **four steps**. Each step adds one idea and builds directly on the previous code. Don't skip ahead — the point is to understand what each piece does before the next one appears.

If you've already done the [Pressure Sensor → Screen](class06-pressure_map_canvas.md) walkthrough, this follows the same incremental structure — but instead of drawing shapes on a canvas, you're controlling a **pre-made animation**.

---

## Step 1 — Make Your Animation

**Goal:** Create a short animation in the LED Matrix Editor and export it as a `.h` file you can use in Arduino.

### Open the Editor

Go to the [Arduino LED Matrix Editor](https://ledmatrix-editor.arduino.cc/). You'll see a 12×8 grid — this matches the LED matrix on your Arduino UNO R4 WiFi.

### Create Your Frames

1. **Paint pixels** by clicking on the grid. Each lit pixel = one LED on the matrix.
2. When you're happy with the first frame, click the **+** button (or press `Ctrl + N`) to add a new frame.
3. To make animation easier, **duplicate the previous frame** (`Ctrl + D`) and modify it — this way each frame is a small change from the last.
4. Use **Shift + Arrow keys** to move all pixels in a frame as a group.
5. Set the **display time** for each frame (how long it stays on screen before advancing).
6. Press **Spacebar** to preview your animation. Press **Shift + Spacebar** to preview with looping.

For a full list of keyboard shortcuts and editor features, see the [LED Matrix Editor Guide](https://digitalfuturesocadu.github.io/TinyFilmFestival/docs/#editor-guide).

### Export the Animation

1. Click the **</>** button to download your animation as a `.h` file.
2. **Name the file with no spaces** — for example `myAnimation` or `walkCycle`.
3. Locate the folder containing your Arduino sketch (the `.ino` file).
4. **Copy the `.h` file into that same folder.** It must be in the same folder as the sketch.

> **Tip:** If you don't want to make your own animation yet, you can download one from the [exampleAnimations folder](https://github.com/DigitalFuturesOCADU/TinyFilmFestival/tree/main/exampleAnimations) — try `landscape.h` for this walkthrough.

### Save Your Project

The editor is web-based — **your work is not saved automatically**. Use `Ctrl + S` to download a `.mpj` project file. You can reload it later with the Load button. The `.h` export is for Arduino; the `.mpj` is for continuing to edit.

---

## Step 2 — Play the Animation

**Goal:** Confirm the animation file works by playing it on the LED matrix with no sensor attached.

```cpp
// ============================================================
// Step 2 — Play your animation on the LED matrix
// ============================================================
// What this does:
//   Loads a .h animation file and plays it on loop.
//   No sensor — just verifying the animation works.
//
// Before uploading:
//   Make sure your .h file is in the same folder
//   as this .ino sketch.
// ============================================================

#include "TinyFilmFestival.h"
#include "landscape.h"          // ← replace with YOUR .h filename

TinyScreen screen;
Animation landscapeAnim = landscape;  // the variable inside landscape.h is called 'landscape'

void setup()
{
    screen.begin();
    screen.play(landscapeAnim, LOOP);   // start playing on loop
}

void loop()
{
    screen.update();           // REQUIRED — advances the animation each frame
}
```

### What to check

1. Upload the sketch. You should see your animation playing on the LED matrix.
2. If nothing appears, double-check that the `.h` file is in the **same folder** as the `.ino` file.
3. The variable name inside the `.h` file **matches the filename**. So `landscape.h` contains a variable called `landscape`, and you write `Animation landscapeAnim = landscape;`. If you export your own file called `walkCycle.h`, the variable inside will be `walkCycle`, and you'd write `Animation walkCycleAnim = walkCycle;`.

> **Note:** `screen.update()` must be called every pass through `loop()`. It's what advances the animation to the next frame at the right time. Without it, the animation freezes on frame 1.

---

## Step 3 — Read the Sensor

**Goal:** Add the pressure sensor and print values to Serial Monitor — but don't connect it to the animation yet.

This is the same sensor setup from [Pressure to Screen — Step 1](class06-pressure_map_canvas.md#step-1--read-the-sensor). The animation keeps playing independently while the sensor prints values in the background.

```cpp
// ============================================================
// Step 3 — Animation plays while sensor prints to Serial
// ============================================================
// What's new:
//   - Pressure sensor on A0 added (no library needed)
//   - Sensor reads every 20 ms and prints to Serial Monitor
//   - The animation still plays on its own — not connected yet
//
// Why this step exists:
//   You need to see your sensor's actual range before you
//   can map it to anything. Press and release the sensor
//   and write down the lightest and hardest press values.
// ============================================================

#include "TinyFilmFestival.h"
#include "landscape.h"          // ← replace with YOUR .h filename

TinyScreen screen;
Animation landscapeAnim = landscape;

// --- Pin configuration ---
int FSR_PIN = A0;

// --- Timing ---
int READ_INTERVAL = 20;
unsigned long lastRead = 0;

// --- Sensor value ---
int pressureReading = 0;       // raw analog value (0–1023)

void setup()
{
    Serial.begin(9600);
    screen.begin();
    screen.play(landscapeAnim, LOOP);
}

void loop()
{
    // --- Sensor reading (printing, not connected to animation) ---
    if (millis() - lastRead >= READ_INTERVAL)
    {
        lastRead = millis();
        pressureReading = analogRead(FSR_PIN);

        Serial.print("Pressure: ");
        Serial.println(pressureReading);
    }

    // --- Animation keeps playing independently ---
    screen.update();
}
```

### What to do with this

1. Upload and open the **Serial Monitor** (9600 baud).
2. The animation plays on the matrix while pressure values scroll in the monitor.
3. Press and release the sensor with varying amounts of force.
4. **Write down two numbers:**
   - The **lightest press** value you see reliably — this is your sensor minimum (it won't be 0)
   - The **hardest press** value before it stops increasing — this is your sensor maximum (it usually won't reach 1023)

These become `LIGHTEST_PRESS` and `HARDEST_PRESS` in the next step. Every custom FSR is different depending on materials, size, and construction, so **measure yours — don't guess**.

---

## Step 4 — Connect Sensor to Animation Speed

**Goal:** Use `map()` to convert pressure into a speed value, then pass it to `screen.setSpeed()` so the animation plays faster when you press harder.

`setSpeed()` accepts either **milliseconds per frame** (integer) or a **multiplier** (float, where `1.0` = original speed, `2.0` = double speed, `0.5` = half speed). We'll use the multiplier approach so the numbers are easier to reason about.

```cpp
// ============================================================
// Step 4 — Pressure controls animation speed
// ============================================================
// What changed:
//   - Variables renamed to describe the interaction:
//     gripStrength, LIGHTEST_PRESS, HARDEST_PRESS,
//     SLOWEST_SPEED, FASTEST_SPEED, playbackSpeed
//   - map() converts pressure → speed percentage
//   - Division by 100.0 converts to the float multiplier
//     that setSpeed() expects
//   - Harder press = faster animation
// ============================================================

#include "TinyFilmFestival.h"
#include "landscape.h"          // ← replace with YOUR .h filename

TinyScreen screen;
Animation landscapeAnim = landscape;

// --- Pin configuration ---
int FSR_PIN = A0;

// --- Sensor range (from YOUR Serial Monitor observations) ---
int LIGHTEST_PRESS = 50;      // update with your real minimum
int HARDEST_PRESS = 800;      // update with your real maximum

// --- Speed range (as percentages of original speed) ---
// We map to integers (50–300) then divide by 100.0 to get a float.
// This avoids dealing with float mapping directly.
int SLOWEST_SPEED = 50;       // 50% — half speed when barely pressing
int FASTEST_SPEED = 300;      // 300% — triple speed when pressing hard

// --- Timing ---
int READ_INTERVAL = 20;
unsigned long lastRead = 0;

// --- Values ---
int gripStrength = 0;          // how hard the sensor is being pressed (raw analog)
int speedPercent = 100;        // mapped speed as a percentage
float playbackSpeed = 1.0;    // what we pass to setSpeed()

void setup()
{
    Serial.begin(9600);
    screen.begin();
    screen.play(landscapeAnim, LOOP);
}

void loop()
{
    if (millis() - lastRead >= READ_INTERVAL)
    {
        lastRead = millis();

        // Read how hard the sensor is being pressed
        gripStrength = analogRead(FSR_PIN);

        // ★ THE CONNECTION — pressure drives the animation speed
        // Harder press = higher speed percentage = faster playback
        speedPercent = map(gripStrength, LIGHTEST_PRESS, HARDEST_PRESS, SLOWEST_SPEED, FASTEST_SPEED);
        speedPercent = constrain(speedPercent, SLOWEST_SPEED, FASTEST_SPEED);

        // Convert percentage to the float multiplier setSpeed() expects
        // 50 → 0.5, 100 → 1.0, 300 → 3.0
        playbackSpeed = speedPercent / 100.0;

        screen.setSpeed(playbackSpeed);

        // Debug printing
        Serial.print("Grip strength: ");
        Serial.print(gripStrength);
        Serial.print(" → Speed: ");
        Serial.print(playbackSpeed);
        Serial.println("x");
    }

    screen.update();
}
```

### Tuning the Interaction

| What to change | Where | Effect |
|---|---|---|
| **Sensor range** | `LIGHTEST_PRESS` / `HARDEST_PRESS` | Narrower range = more sensitive. Use the real values from Step 3. |
| **Speed range** | `SLOWEST_SPEED` / `FASTEST_SPEED` | `50` = half speed, `100` = normal, `200` = double, `300` = triple. Try `25` for very slow. |
| **Map direction** | Swap `FASTEST_SPEED` and `SLOWEST_SPEED` in `map()` | Reverses the interaction — harder press = slower instead of faster. |
| **Read speed** | `READ_INTERVAL` | Lower = more responsive speed changes. Higher = smoother but slower to react. |
| **Play mode** | `LOOP` in `screen.play()` | Try `BOOMERANG` for ping-pong playback, or `ONCE` to play a single pass and stop. |

### What's Next?

You now have the core pattern: **read → translate → apply**. From here you can control other animation properties with the same sensor. The examples below show how.

---

## Going Further

The examples below all start from the Step 4 code. Each one changes what the sensor controls — position, layers, inversion — using the same **read → translate → apply** pattern. Comments highlight what's different.

---

### Example: Pressure Controls Animation Position

Instead of controlling **speed**, the sensor controls **where the animation appears** on the matrix. The animation slides left and right as you press harder.

`setPosition(x, y)` shifts the animation by an offset — pixels that move beyond the 12×8 edges are automatically clipped.

```cpp
#include "TinyFilmFestival.h"
#include "landscape.h"          // ← replace with YOUR .h filename

TinyScreen screen;
Animation landscapeAnim = landscape;

int FSR_PIN = A0;

// --- Sensor range ---
int LIGHTEST_PRESS = 50;
int HARDEST_PRESS = 800;

// --- Position range ---
// Named for what they mean on screen
int LEFT_OFFSET = -4;     // shift animation left (partially off-screen)
int RIGHT_OFFSET = 4;     // shift animation right (partially off-screen)

int READ_INTERVAL = 20;
unsigned long lastRead = 0;

int gripStrength = 0;          // how hard the sensor is being pressed
int animationX = 0;            // horizontal offset for the animation

void setup()
{
    Serial.begin(9600);
    screen.begin();
    screen.play(landscapeAnim, LOOP);
}

void loop()
{
    if (millis() - lastRead >= READ_INTERVAL)
    {
        lastRead = millis();
        gripStrength = analogRead(FSR_PIN);

        // Harder press = animation shifts right, lighter = shifts left
        animationX = map(gripStrength, LIGHTEST_PRESS, HARDEST_PRESS, LEFT_OFFSET, RIGHT_OFFSET);
        animationX = constrain(animationX, LEFT_OFFSET, RIGHT_OFFSET);

        // Apply the position offset
        screen.setPosition(animationX, 0);

        Serial.print("Grip strength: ");
        Serial.print(gripStrength);
        Serial.print(" → X offset: ");
        Serial.println(animationX);
    }

    screen.update();
}
```

**What changed:** `map()` now feeds `animationX` and passes it to `screen.setPosition()` instead of `screen.setSpeed()`. The animation still plays at its original speed — only its position moves. Negative offsets shift left (clipping at the edge), positive shift right.

---

### Example: Pressure Controls Vertical Position

Same idea, but the animation moves **up and down** instead of left and right.

```cpp
#include "TinyFilmFestival.h"
#include "landscape.h"

TinyScreen screen;
Animation landscapeAnim = landscape;

int FSR_PIN = A0;

int LIGHTEST_PRESS = 50;
int HARDEST_PRESS = 800;

// --- Vertical position range ---
int TOP_OFFSET = -3;      // push animation up (partially off-screen)
int BOTTOM_OFFSET = 3;    // push animation down (partially off-screen)

int READ_INTERVAL = 20;
unsigned long lastRead = 0;

int gripStrength = 0;
int animationY = 0;            // vertical offset for the animation

void setup()
{
    Serial.begin(9600);
    screen.begin();
    screen.play(landscapeAnim, LOOP);
}

void loop()
{
    if (millis() - lastRead >= READ_INTERVAL)
    {
        lastRead = millis();
        gripStrength = analogRead(FSR_PIN);

        // Harder press = animation moves up, lighter = moves down
        animationY = map(gripStrength, LIGHTEST_PRESS, HARDEST_PRESS, BOTTOM_OFFSET, TOP_OFFSET);
        animationY = constrain(animationY, TOP_OFFSET, BOTTOM_OFFSET);

        screen.setPosition(0, animationY);
    }

    screen.update();
}
```

**What changed:** The position offset is on the **Y axis** instead of X. `screen.setPosition(0, animationY)` keeps horizontal position at 0 and moves the animation vertically. Harder press = up (lower Y), lighter = down (higher Y).

---

### Example: Pressure Controls Speed AND Position

The sensor drives **two properties at once** — the animation speeds up and slides right as you press harder.

```cpp
#include "TinyFilmFestival.h"
#include "landscape.h"

TinyScreen screen;
Animation landscapeAnim = landscape;

int FSR_PIN = A0;

int LIGHTEST_PRESS = 50;
int HARDEST_PRESS = 800;

// --- Speed range (as percentage) ---
int SLOWEST_SPEED = 50;
int FASTEST_SPEED = 300;

// --- Position range ---
int LEFT_OFFSET = -4;
int RIGHT_OFFSET = 4;

int READ_INTERVAL = 20;
unsigned long lastRead = 0;

int gripStrength = 0;
int speedPercent = 100;
float playbackSpeed = 1.0;
int animationX = 0;

void setup()
{
    Serial.begin(9600);
    screen.begin();
    screen.play(landscapeAnim, LOOP);
}

void loop()
{
    if (millis() - lastRead >= READ_INTERVAL)
    {
        lastRead = millis();
        gripStrength = analogRead(FSR_PIN);

        // Speed: harder press = faster
        speedPercent = map(gripStrength, LIGHTEST_PRESS, HARDEST_PRESS, SLOWEST_SPEED, FASTEST_SPEED);
        speedPercent = constrain(speedPercent, SLOWEST_SPEED, FASTEST_SPEED);
        playbackSpeed = speedPercent / 100.0;
        screen.setSpeed(playbackSpeed);

        // Position: harder press = further right
        animationX = map(gripStrength, LIGHTEST_PRESS, HARDEST_PRESS, LEFT_OFFSET, RIGHT_OFFSET);
        animationX = constrain(animationX, LEFT_OFFSET, RIGHT_OFFSET);
        screen.setPosition(animationX, 0);
    }

    screen.update();
}
```

**What changed:** Two `map()` calls from the same sensor — one for speed, one for position. Same pattern as the [Two Shapes example](class06-pressure_map_canvas.md#example-two-shapes-from-one-sensor) in the canvas walkthrough. You can map one sensor to as many animation parameters as you want.

---

### Example: Two Layered Animations at Different Speeds

Stack two animations on top of each other. The sensor controls the **foreground speed** while the background plays at a fixed rate. Lit pixels on the foreground layer appear on top of the background.

```cpp
#include "TinyFilmFestival.h"
#include "landscape.h"          // background animation
#include "go.h"                 // foreground animation — use a different .h file

TinyScreen screen;

// Each .h file's variable matches its filename:
// landscape.h → landscape, go.h → go
Animation backgroundAnim = landscape;
Animation foregroundAnim = go;

int FSR_PIN = A0;

int LIGHTEST_PRESS = 50;
int HARDEST_PRESS = 800;

// --- Foreground speed range ---
int SLOWEST_FOREGROUND = 30;    // 30% speed when barely pressing
int FASTEST_FOREGROUND = 400;   // 4x speed when pressing hard

int READ_INTERVAL = 20;
unsigned long lastRead = 0;

int gripStrength = 0;
int speedPercent = 100;
float foregroundSpeed = 1.0;

void setup()
{
    Serial.begin(9600);
    screen.begin();

    // Layer 0 = background (plays at original speed)
    screen.play(backgroundAnim, LOOP);

    // Add a second layer and play the foreground animation on it
    screen.addLayer();
    screen.playOnLayer(1, foregroundAnim, LOOP);
}

void loop()
{
    if (millis() - lastRead >= READ_INTERVAL)
    {
        lastRead = millis();
        gripStrength = analogRead(FSR_PIN);

        // Only the foreground speed changes — background stays constant
        speedPercent = map(gripStrength, LIGHTEST_PRESS, HARDEST_PRESS, SLOWEST_FOREGROUND, FASTEST_FOREGROUND);
        speedPercent = constrain(speedPercent, SLOWEST_FOREGROUND, FASTEST_FOREGROUND);
        foregroundSpeed = speedPercent / 100.0;

        screen.setSpeedOnLayer(1, foregroundSpeed);
    }

    screen.update();
}
```

**What changed:** `screen.addLayer()` creates a second animation layer. The background plays on layer 0 at its original speed. The foreground plays on layer 1 with its speed controlled by the sensor via `screen.setSpeedOnLayer(1, ...)`. Lit pixels on the foreground layer are drawn on top of the background. Each `.h` file exports a variable matching its filename (`landscape` and `go`), so there's no naming conflict.

---

### Example: Pressure Inverts the Display

At a threshold pressure, the entire display **inverts** — all lit pixels turn off and all off pixels turn on. This creates a dramatic visual flip with a single API call.

```cpp
#include "TinyFilmFestival.h"
#include "landscape.h"

TinyScreen screen;
Animation landscapeAnim = landscape;

int FSR_PIN = A0;

int LIGHTEST_PRESS = 50;
int HARDEST_PRESS = 800;

// --- Threshold ---
// When grip crosses this value, the display inverts
int INVERT_THRESHOLD = 400;   // roughly halfway through the usable range

int READ_INTERVAL = 20;
unsigned long lastRead = 0;

int gripStrength = 0;

void setup()
{
    Serial.begin(9600);
    screen.begin();
    screen.play(landscapeAnim, LOOP);
}

void loop()
{
    if (millis() - lastRead >= READ_INTERVAL)
    {
        lastRead = millis();
        gripStrength = analogRead(FSR_PIN);

        // Invert when pressing harder than the threshold
        if (gripStrength > INVERT_THRESHOLD)
        {
            screen.setInvert(true);
        }
        else
        {
            screen.setInvert(false);
        }
    }

    screen.update();
}
```

**What changed:** No `map()` at all — this is a **threshold** interaction instead of a continuous mapping. `setInvert(true)` flips every pixel on the display. Combined with a playing animation, the visual effect is immediate and dramatic. You could combine this with speed or position control from the previous examples.

---

### Example: Oscillating Position with Sensor-Controlled Speed

The animation oscillates back and forth automatically using `oscillateInt()`, but the **speed of the oscillation** is controlled by pressure. This combines the animation's own playback with an additional layer of motion.

```cpp
#include "TinyFilmFestival.h"
#include "landscape.h"

TinyScreen screen;
Animation landscapeAnim = landscape;

int FSR_PIN = A0;

int LIGHTEST_PRESS = 50;
int HARDEST_PRESS = 800;

// --- Oscillation range ---
int LEFT_LIMIT = -3;       // furthest left the animation slides
int RIGHT_LIMIT = 3;       // furthest right

// --- Oscillation speed range ---
int SLOWEST_SLIDE = 4000;   // ms — slow drift when barely pressing
int FASTEST_SLIDE = 500;    // ms — fast slide when pressing hard

int READ_INTERVAL = 20;
unsigned long lastRead = 0;

int gripStrength = 0;
int slideSpeed = 2000;         // ms — period of the oscillation

void setup()
{
    Serial.begin(9600);
    screen.begin();
    screen.play(landscapeAnim, LOOP);
}

void loop()
{
    if (millis() - lastRead >= READ_INTERVAL)
    {
        lastRead = millis();
        gripStrength = analogRead(FSR_PIN);

        // Map pressure to oscillation period
        // Harder press = smaller period = faster slide
        slideSpeed = map(gripStrength, LIGHTEST_PRESS, HARDEST_PRESS, SLOWEST_SLIDE, FASTEST_SLIDE);
        slideSpeed = constrain(slideSpeed, FASTEST_SLIDE, SLOWEST_SLIDE);
    }

    // oscillateInt handles the back-and-forth motion
    // slideSpeed controls how fast it completes one cycle
    int animX = oscillateInt(LEFT_LIMIT, RIGHT_LIMIT, slideSpeed);
    screen.setPosition(animX, 0);

    screen.update();
}
```

**What changed:** The sensor doesn't control `setPosition()` directly — it controls `slideSpeed`, which sets how fast `oscillateInt()` completes its cycle. The animation slides back and forth on its own; the sensor controls *how urgently* it slides. This gives you two layers of motion: the animation's own frame playback, plus the physical sliding across the matrix.

> **Combine axes:** Add a second `oscillateInt()` for Y with a different period or a [phase offset](https://digitalfuturesocadu.github.io/TinyFilmFestival/docs/#animation-mode) to create diagonal or circular motion paths:
> ```cpp
> int animX = oscillateInt(-3, 3, slideSpeed);
> int animY = oscillateInt(-2, 2, slideSpeed * 1.5, 0.25);  // phase-shifted
> screen.setPosition(animX, animY);
> ```

# Pressure Thresholds → Animation: Step by Step

[← Back to Class 06](class06-Feb13.md)

This walkthrough takes you from a blank animation to a **threshold-controlled animation** in **five steps**. Instead of smoothly mapping a sensor value to a single property (like the [Pressure → Animation](class06-pressure_map_animation.md) walkthrough does), here you **divide the sensor range into zones** and change what the animation does in each zone.

The scenario: the pressure sensor is built into a handheld object. The animation changes behavior depending on how hard the person is **squeezing** — a light touch, a moderate grip, or a firm press.

If you've already done the [Pressure Thresholds → Shapes](class06-pressure_threshold_canvas.md) walkthrough, this follows the same threshold approach — but instead of drawing on a canvas, you're controlling a **pre-made animation**.

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

Wire the pressure sensor as shown in the [Class 06 wiring diagram](class06-Feb13.md#pressure-sensor--how-does-it-work). The animation keeps playing independently while the sensor prints values in the background.

```cpp
// ============================================================
// Step 3 — Animation plays while sensor prints to Serial
// ============================================================
// What's new:
//   - Pressure sensor wired to A0 with a 10kΩ pull-down resistor
//   - analogRead() reads the sensor every 20 ms
//   - Prints raw value (0–1023) to Serial Monitor
//   - The animation still plays on its own — not connected yet
//
// Why this step exists:
//   You need to see your sensor's actual range before you
//   can define threshold boundaries. Squeeze the sensor at
//   different pressures and write down the values.
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
int gripStrength = 0;

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
        gripStrength = analogRead(FSR_PIN);

        Serial.print("Grip strength: ");
        Serial.println(gripStrength);
    }

    // --- Animation keeps playing independently ---
    screen.update();
}
```

### What to do with this

1. Upload and open the **Serial Monitor** (9600 baud).
2. The animation plays on the matrix while pressure values scroll in the monitor.
3. Squeeze the sensor at different intensities — light, moderate, hard.
4. **Write down three things:**
   - The value when you're **barely touching** the sensor — your minimum
   - The value when you're **pressing as hard as you can** — your maximum
   - Roughly where the **middle** of that range falls

You'll use these numbers to define your threshold boundaries in Step 4.

---

## Step 4 — Define Threshold Zones

**Goal:** Split the sensor range into three zones and print which zone the grip is in. The animation still plays independently — not connected yet.

Instead of using `map()` to create a continuous output, we use `if / else if / else` to divide the range into **discrete zones**. Each zone gets a descriptive name: `LIGHT`, `MEDIUM`, `FIRM`.

```cpp
// ============================================================
// Step 4 — Divide the pressure into three zones
// ============================================================
// What's new:
//   - Two threshold boundaries split the range into 3 zones
//   - if / else if / else determines which zone the grip is in
//   - currentZone tracks the zone as a number (1, 2, 3)
//   - Serial Monitor prints both the value and the zone
//   - The animation plays independently — not connected yet
//
// Why thresholds instead of map():
//   Sometimes you don't want a smooth gradient — you want
//   distinct states. "Light touch," "moderate grip," and
//   "firm press" are three different behaviors, not a spectrum.
// ============================================================

#include "TinyFilmFestival.h"
#include "landscape.h"

TinyScreen screen;
Animation landscapeAnim = landscape;

// --- Pin configuration ---
int FSR_PIN = A0;

// --- Threshold boundaries (raw analogRead values, 0–1023) ---
// These divide the sensor range into three zones.
// Adjust based on YOUR Serial Monitor observations from Step 3.
int LIGHT_TO_MEDIUM = 200;   // below this = LIGHT zone (barely touching)
int MEDIUM_TO_FIRM = 600;    // above this = FIRM zone (pressing hard)
                              // between = MEDIUM zone (moderate grip)

// --- Timing ---
int READ_INTERVAL = 20;
unsigned long lastRead = 0;

// --- Values ---
int gripStrength = 0;
int currentZone = 0;          // 1 = LIGHT, 2 = MEDIUM, 3 = FIRM

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

        // Determine which zone the grip is in
        if (gripStrength < LIGHT_TO_MEDIUM)
        {
            currentZone = 1;  // LIGHT — barely touching
        }
        else if (gripStrength < MEDIUM_TO_FIRM)
        {
            currentZone = 2;  // MEDIUM — moderate grip
        }
        else
        {
            currentZone = 3;  // FIRM — pressing hard
        }

        // Print both values so we can verify the zones
        Serial.print("Grip: ");
        Serial.print(gripStrength);
        Serial.print(" → Zone: ");
        Serial.println(currentZone);
    }

    // Animation plays independently — not connected to zones yet
    screen.update();
}
```

### What to notice

- **Two boundaries create three zones.** `LIGHT_TO_MEDIUM` and `MEDIUM_TO_FIRM` are all you need. Everything below the first is LIGHT, everything above the second is FIRM, everything between is MEDIUM.
- **The zone names describe the interaction.** `LIGHT_TO_MEDIUM = 200` tells you that below 200 the grip is considered "light." Compare that to `THRESHOLD_1 = 200` — same number, but no meaning.
- **The animation keeps playing.** Nothing on screen changes based on zone — we're just verifying the zone logic works before connecting it.
- **Tuning:** If the Serial Monitor shows the zone jumping erratically, widen the gap between thresholds or move them to where your actual pressure transitions feel natural.

---

## Step 5 — Connect Zones to Animation Speed

**Goal:** Each zone sets a different playback speed. The animation plays slow with a light touch, normal at a moderate grip, and fast when squeezed hard.

```cpp
// ============================================================
// Step 5 — Zone controls animation speed
// ============================================================
// What changed:
//   Each zone now sets a different playbackSpeed using
//   screen.setSpeed(). The animation reacts with distinct
//   speed jumps at each boundary — not a smooth gradient.
// ============================================================

#include "TinyFilmFestival.h"
#include "landscape.h"

TinyScreen screen;
Animation landscapeAnim = landscape;

// --- Pin configuration ---
int FSR_PIN = A0;

// --- Threshold boundaries (from YOUR observations) ---
int LIGHT_TO_MEDIUM = 200;   // below = barely touching
int MEDIUM_TO_FIRM = 600;    // above = pressing hard

// --- Speed per zone (as float multiplier) ---
// 1.0 = original speed, 2.0 = double, 0.5 = half
float LIGHT_SPEED = 0.3;    // slow — light touch, calm
float MEDIUM_SPEED = 1.0;   // normal — moderate grip
float FIRM_SPEED = 3.0;     // fast — pressing hard, high energy

// --- Timing ---
int READ_INTERVAL = 20;
unsigned long lastRead = 0;

// --- Values ---
int gripStrength = 0;
int currentZone = 0;
float playbackSpeed = 1.0;

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

        // ★ THE CONNECTION — zone determines the speed
        if (gripStrength < LIGHT_TO_MEDIUM)
        {
            currentZone = 1;
            playbackSpeed = LIGHT_SPEED;
        }
        else if (gripStrength < MEDIUM_TO_FIRM)
        {
            currentZone = 2;
            playbackSpeed = MEDIUM_SPEED;
        }
        else
        {
            currentZone = 3;
            playbackSpeed = FIRM_SPEED;
        }

        screen.setSpeed(playbackSpeed);

        Serial.print("Grip: ");
        Serial.print(gripStrength);
        Serial.print(" → Zone ");
        Serial.print(currentZone);
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
| **Zone boundaries** | `LIGHT_TO_MEDIUM` / `MEDIUM_TO_FIRM` | Shift where the speed changes happen |
| **Speed values** | `LIGHT_SPEED` / `MEDIUM_SPEED` / `FIRM_SPEED` | Try `0.0` for a full pause, `5.0` for frantic |
| **Number of zones** | Add more `else if` blocks + more speed values | More zones = more distinct speed levels |
| **Play mode** | `LOOP` in `screen.play()` | Try `BOOMERANG` for ping-pong playback |

### What's Next?

You now have the core pattern: **read → classify → apply**. Each zone sets a fixed property value — the animation jumps between speeds rather than sliding smoothly. From here you can control other animation properties per zone, or switch between entirely different animations. The examples below show how.

---

## Going Further

The examples below all start from the Step 5 code. Each one extends the threshold idea in a different direction. Comments highlight what's different.

---

### Example: Zone Controls Speed and Position

Each zone sets both a **speed** and a **position offset**. The animation speeds up and slides to the right as the grip gets firmer.

```cpp
#include "TinyFilmFestival.h"
#include "landscape.h"

TinyScreen screen;
Animation landscapeAnim = landscape;

int FSR_PIN = A0;

int LIGHT_TO_MEDIUM = 200;
int MEDIUM_TO_FIRM = 600;

// --- Speed per zone ---
float LIGHT_SPEED = 0.3;
float MEDIUM_SPEED = 1.0;
float FIRM_SPEED = 3.0;

// --- Position offset per zone ---
int LIGHT_X = -3;           // shifted left when light
int MEDIUM_X = 0;           // centered
int FIRM_X = 3;             // shifted right when firm

int READ_INTERVAL = 20;
unsigned long lastRead = 0;

int gripStrength = 0;
int currentZone = 0;
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

        if (gripStrength < LIGHT_TO_MEDIUM)
        {
            currentZone = 1;
            playbackSpeed = LIGHT_SPEED;
            animationX = LIGHT_X;
        }
        else if (gripStrength < MEDIUM_TO_FIRM)
        {
            currentZone = 2;
            playbackSpeed = MEDIUM_SPEED;
            animationX = MEDIUM_X;
        }
        else
        {
            currentZone = 3;
            playbackSpeed = FIRM_SPEED;
            animationX = FIRM_X;
        }

        screen.setSpeed(playbackSpeed);
        screen.setPosition(animationX, 0);
    }

    screen.update();
}
```

**What changed:** Each `if` block now sets **two properties** — speed and position. Adding more properties per zone is just adding more lines inside each block. The animation jumps to a new speed *and* a new position at each boundary.

---

### Example: Zone Controls Invert

The display **inverts** at a threshold — all lit pixels turn off and all off pixels turn on. Combined with speed changes, this creates a dramatic shift between zones.

```cpp
#include "TinyFilmFestival.h"
#include "landscape.h"

TinyScreen screen;
Animation landscapeAnim = landscape;

int FSR_PIN = A0;

int LIGHT_TO_MEDIUM = 200;
int MEDIUM_TO_FIRM = 600;

// --- Properties per zone ---
float LIGHT_SPEED = 0.3;
float MEDIUM_SPEED = 1.0;
float FIRM_SPEED = 3.0;

// FIRM zone inverts the display for an "overload" feel
// LIGHT and MEDIUM stay normal

int READ_INTERVAL = 20;
unsigned long lastRead = 0;

int gripStrength = 0;
int currentZone = 0;

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

        if (gripStrength < LIGHT_TO_MEDIUM)
        {
            currentZone = 1;
            screen.setSpeed(LIGHT_SPEED);
            screen.setInvert(false);      // normal
        }
        else if (gripStrength < MEDIUM_TO_FIRM)
        {
            currentZone = 2;
            screen.setSpeed(MEDIUM_SPEED);
            screen.setInvert(false);      // normal
        }
        else
        {
            currentZone = 3;
            screen.setSpeed(FIRM_SPEED);
            screen.setInvert(true);       // inverted — squeezing hard
        }
    }

    screen.update();
}
```

**What changed:** `screen.setInvert(true)` flips every pixel on the display. The FIRM zone inverts and speeds up — it feels like overload. LIGHT and MEDIUM stay normal. You could invert any zone, or combine invert with position changes for an even stronger effect.

---

### Example: Zone Switches Between Animations

Instead of changing properties of one animation, each zone **plays a different animation entirely**. Squeezing harder switches which animation is on screen.

> Download animations from the [exampleAnimations folder](https://github.com/DigitalFuturesOCADU/TinyFilmFestival/tree/main/exampleAnimations) or create your own with the [LED Matrix Editor](https://ledmatrix-editor.arduino.cc/). See the [Editor Guide](https://digitalfuturesocadu.github.io/TinyFilmFestival/docs/#editor-guide) for how to export `.h` files.

```cpp
#include "TinyFilmFestival.h"
#include "idle.h"
#include "fiz.h"
#include "landscape.h"

TinyScreen screen;

// Each .h file's variable matches its filename
Animation idleAnim = idle;
Animation fizAnim = fiz;
Animation landscapeAnim = landscape;

int FSR_PIN = A0;

int LIGHT_TO_MEDIUM = 200;
int MEDIUM_TO_FIRM = 600;

int READ_INTERVAL = 20;
unsigned long lastRead = 0;

int gripStrength = 0;
int currentZone = 0;
int lastZone = 0;             // tracks previous zone to avoid restarting

void setup()
{
    Serial.begin(9600);
    screen.begin();
    screen.play(idleAnim, LOOP);   // start with LIGHT animation
    lastZone = 1;
}

void loop()
{
    if (millis() - lastRead >= READ_INTERVAL)
    {
        lastRead = millis();
        gripStrength = analogRead(FSR_PIN);

        if (gripStrength < LIGHT_TO_MEDIUM)
        {
            currentZone = 1;
        }
        else if (gripStrength < MEDIUM_TO_FIRM)
        {
            currentZone = 2;
        }
        else
        {
            currentZone = 3;
        }

        // Only change animation when we enter a NEW zone
        // Without this check, screen.play() restarts from frame 1
        // every pass through loop() — and the animation never advances
        if (currentZone != lastZone)
        {
            if (currentZone == 1) screen.play(idleAnim, LOOP);
            else if (currentZone == 2) screen.play(fizAnim, LOOP);
            else screen.play(landscapeAnim, LOOP);

            lastZone = currentZone;
        }
    }

    screen.update();
}
```

**What changed:** Instead of changing speed or position, each zone calls `screen.play()` with a different animation. The `lastZone` check is critical — without it, `screen.play()` would restart the animation from frame 1 on every pass through `loop()`, and you'd never see it advance past the first frame. Each `.h` file exports a variable matching its filename (`idle`, `fiz`, `landscape`), so there's no naming conflict.

---

### Example: Switch Animations with Speed Per Zone

Combine switching animations **and** setting a speed for each. Each zone gets its own animation playing at its own pace.

```cpp
#include "TinyFilmFestival.h"
#include "idle.h"
#include "fiz.h"
#include "landscape.h"

TinyScreen screen;

Animation idleAnim = idle;
Animation fizAnim = fiz;
Animation landscapeAnim = landscape;

int FSR_PIN = A0;

int LIGHT_TO_MEDIUM = 200;
int MEDIUM_TO_FIRM = 600;

// --- Speed per zone ---
float LIGHT_SPEED = 0.5;      // slow playback for the LIGHT animation
float MEDIUM_SPEED = 1.0;     // normal for MEDIUM
float FIRM_SPEED = 2.5;       // fast for FIRM

int READ_INTERVAL = 20;
unsigned long lastRead = 0;

int gripStrength = 0;
int currentZone = 0;
int lastZone = 0;

void setup()
{
    Serial.begin(9600);
    screen.begin();
    screen.play(idleAnim, LOOP);
    screen.setSpeed(LIGHT_SPEED);
    lastZone = 1;
}

void loop()
{
    if (millis() - lastRead >= READ_INTERVAL)
    {
        lastRead = millis();
        gripStrength = analogRead(FSR_PIN);

        if (gripStrength < LIGHT_TO_MEDIUM)
        {
            currentZone = 1;
        }
        else if (gripStrength < MEDIUM_TO_FIRM)
        {
            currentZone = 2;
        }
        else
        {
            currentZone = 3;
        }

        // Switch animation AND speed when entering a new zone
        if (currentZone != lastZone)
        {
            if (currentZone == 1)
            {
                screen.play(idleAnim, LOOP);
                screen.setSpeed(LIGHT_SPEED);
            }
            else if (currentZone == 2)
            {
                screen.play(fizAnim, LOOP);
                screen.setSpeed(MEDIUM_SPEED);
            }
            else
            {
                screen.play(landscapeAnim, LOOP);
                screen.setSpeed(FIRM_SPEED);
            }

            lastZone = currentZone;
        }
    }

    screen.update();
}
```

**What changed:** Each zone block now calls both `screen.play()` and `screen.setSpeed()`. The speed is set inside the `lastZone` check so it only changes when the zone changes. You could add `screen.setPosition()` or `screen.setInvert()` here too — each zone becomes a full "scene" with its own animation and properties.

---

### Example: Two Layered Animations

Stack two animations on top of each other. The zone controls the **foreground speed** while the background plays at a fixed rate. Lit pixels on the foreground layer appear on top of the background.

```cpp
#include "TinyFilmFestival.h"
#include "landscape.h"          // background animation
#include "go.h"                 // foreground animation

TinyScreen screen;

// Each .h file's variable matches its filename
Animation backgroundAnim = landscape;
Animation foregroundAnim = go;

int FSR_PIN = A0;

int LIGHT_TO_MEDIUM = 200;
int MEDIUM_TO_FIRM = 600;

// --- Foreground speed per zone ---
float LIGHT_FOREGROUND = 0.2;      // barely moving when light
float MEDIUM_FOREGROUND = 1.0;     // normal
float FIRM_FOREGROUND = 4.0;       // fast overlay when firm

int READ_INTERVAL = 20;
unsigned long lastRead = 0;

int gripStrength = 0;
int currentZone = 0;
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

        if (gripStrength < LIGHT_TO_MEDIUM)
        {
            currentZone = 1;
            foregroundSpeed = LIGHT_FOREGROUND;
        }
        else if (gripStrength < MEDIUM_TO_FIRM)
        {
            currentZone = 2;
            foregroundSpeed = MEDIUM_FOREGROUND;
        }
        else
        {
            currentZone = 3;
            foregroundSpeed = FIRM_FOREGROUND;
        }

        // Only the foreground speed changes — background stays constant
        screen.setSpeedOnLayer(1, foregroundSpeed);
    }

    screen.update();
}
```

**What changed:** `screen.addLayer()` creates a second animation layer. The background plays on layer 0 at its original speed. The foreground plays on layer 1 with its speed jumping between zone values via `screen.setSpeedOnLayer(1, ...)`. Lit pixels on the foreground layer are drawn on top of the background. Each `.h` file exports a variable matching its filename (`landscape` and `go`), so there's no naming conflict.

---

### Example: Oscillating Position Per Zone

The animation oscillates back and forth using `oscillateInt()`, but each zone sets a **different oscillation speed**. LIGHT drifts gently, MEDIUM moves at a moderate pace, FIRM jitters fast.

```cpp
#include "TinyFilmFestival.h"
#include "landscape.h"

TinyScreen screen;
Animation landscapeAnim = landscape;

int FSR_PIN = A0;

int LIGHT_TO_MEDIUM = 200;
int MEDIUM_TO_FIRM = 600;

// --- Oscillation range ---
int LEFT_LIMIT = -3;
int RIGHT_LIMIT = 3;

// --- Oscillation speed per zone (ms per cycle) ---
int LIGHT_SLIDE = 4000;       // barely moving
int MEDIUM_SLIDE = 1500;      // gentle drift
int FIRM_SLIDE = 400;         // fast jitter

int READ_INTERVAL = 20;
unsigned long lastRead = 0;

int gripStrength = 0;
int currentZone = 0;
int slideSpeed = 1500;

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

        if (gripStrength < LIGHT_TO_MEDIUM)
        {
            currentZone = 1;
            slideSpeed = LIGHT_SLIDE;
        }
        else if (gripStrength < MEDIUM_TO_FIRM)
        {
            currentZone = 2;
            slideSpeed = MEDIUM_SLIDE;
        }
        else
        {
            currentZone = 3;
            slideSpeed = FIRM_SLIDE;
        }
    }

    // oscillateInt handles the back-and-forth motion
    // slideSpeed jumps at each zone boundary
    int animX = oscillateInt(LEFT_LIMIT, RIGHT_LIMIT, slideSpeed);
    screen.setPosition(animX, 0);

    screen.update();
}
```

**What changed:** Instead of the sensor directly controlling position, each zone sets a fixed `slideSpeed` for `oscillateInt()`. The animation slides back and forth on its own — the zone controls *how urgently* it slides. This gives you two layers of motion: the animation's own frame playback, plus the physical sliding across the matrix. The speed jumps at zone boundaries rather than changing smoothly.

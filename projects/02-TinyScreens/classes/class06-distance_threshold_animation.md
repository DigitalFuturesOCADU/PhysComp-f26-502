# Distance Thresholds → Animation: Step by Step

[← Back to Class 06](class06-Feb13.md)

This walkthrough takes you from a blank animation to a **threshold-controlled animation** in **five steps**. Instead of smoothly mapping a sensor value to a single property (like the [Distance → Animation](class06-distance_map_animation.md) walkthrough does), here you **divide the sensor range into zones** and change what the animation does in each zone.

The scenario: the distance sensor is mounted on an object pointing outward, measuring **how far away the viewer's face is**. The animation changes behavior depending on whether the person is leaning in close, standing at a normal distance, or far away.

If you've already done the [Distance Thresholds → Shapes](class06-distance_threshold_canvas.md) walkthrough, this follows the same threshold approach — but instead of drawing on a canvas, you're controlling a **pre-made animation**.

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

**Goal:** Add the distance sensor and print values to Serial Monitor — but don't connect it to the animation yet.

Wire the HC-SR04 as shown in the [Class 06 wiring diagram](class06-Feb13.md#distance-sensor--how-does-it-work). The animation keeps playing independently while the sensor prints values in the background.

```cpp
// ============================================================
// Step 3 — Animation plays while sensor prints to Serial
// ============================================================
// What's new:
//   - EasyUltrasonic library and sensor wiring added
//   - Sensor reads every 20 ms and prints to Serial Monitor
//   - The animation still plays on its own — not connected yet
//
// Why this step exists:
//   You need to see your sensor's actual range before you
//   can define threshold boundaries. Have someone lean toward
//   and away from the sensor and write down the values.
// ============================================================

#include "TinyFilmFestival.h"
#include <EasyUltrasonic.h>
#include "landscape.h"          // ← replace with YOUR .h filename

TinyScreen screen;
EasyUltrasonic ultrasonic;
Animation landscapeAnim = landscape;

// --- Pin configuration ---
int TRIG_PIN = A0;
int ECHO_PIN = A1;

// --- Sensor range ---
int MIN_DISTANCE = 2;
int MAX_DISTANCE = 100;

// --- Timing ---
int READ_INTERVAL = 20;
unsigned long lastRead = 0;

// --- Sensor value ---
float distanceToFace = 0;

void setup()
{
    Serial.begin(9600);
    screen.begin();
    ultrasonic.attach(TRIG_PIN, ECHO_PIN, MIN_DISTANCE, MAX_DISTANCE);
    screen.play(landscapeAnim, LOOP);
}

void loop()
{
    // --- Sensor reading (printing, not connected to animation) ---
    if (millis() - lastRead >= READ_INTERVAL)
    {
        lastRead = millis();
        distanceToFace = ultrasonic.getDistanceCM();

        Serial.print("Distance to face (cm): ");
        Serial.println(distanceToFace);
    }

    // --- Animation keeps playing independently ---
    screen.update();
}
```

### What to do with this

1. Upload and open the **Serial Monitor** (9600 baud).
2. The animation plays on the matrix while distance values scroll in the monitor.
3. Move your face toward and away from the sensor.
4. **Write down three things:**
   - The value when your face is **very close** — your minimum
   - The value when your face is at **arm's length** — your maximum
   - Roughly where the **middle** of that range falls

You'll use these numbers to define your threshold boundaries in Step 4.

---

## Step 4 — Define Threshold Zones

**Goal:** Split the sensor range into three zones and print which zone the face is in. The animation still plays independently — not connected yet.

Instead of using `map()` to create a continuous output, we use `if / else if / else` to divide the range into **discrete zones**. Each zone gets a descriptive name: `NEAR`, `MIDDLE`, `FAR`.

```cpp
// ============================================================
// Step 4 — Divide the distance into three zones
// ============================================================
// What's new:
//   - Two threshold boundaries split the range into 3 zones
//   - if / else if / else determines which zone the face is in
//   - currentZone tracks the zone as a number (1, 2, 3)
//   - Serial Monitor prints both the distance and the zone
//   - The animation plays independently — not connected yet
//
// Why thresholds instead of map():
//   Sometimes you don't want a smooth gradient — you want
//   distinct states. "Leaning in," "standing normally," and
//   "far away" are three different behaviors, not a spectrum.
// ============================================================

#include "TinyFilmFestival.h"
#include <EasyUltrasonic.h>
#include "landscape.h"

TinyScreen screen;
EasyUltrasonic ultrasonic;
Animation landscapeAnim = landscape;

// --- Pin configuration ---
int TRIG_PIN = A0;
int ECHO_PIN = A1;

// --- Sensor range ---
int MIN_DISTANCE = 2;
int MAX_DISTANCE = 80;

// --- Threshold boundaries (cm) ---
// These divide the sensor range into three zones.
// Adjust based on YOUR Serial Monitor observations from Step 3.
int NEAR_TO_MID = 20;       // below this = NEAR zone (leaning in)
int MID_TO_FAR = 50;        // above this = FAR zone (standing back)
                              // between = MIDDLE zone (normal viewing)

// --- Timing ---
int READ_INTERVAL = 20;
unsigned long lastRead = 0;

// --- Values ---
float distanceToFace = 0;
int currentZone = 0;          // 1 = NEAR, 2 = MIDDLE, 3 = FAR

void setup()
{
    Serial.begin(9600);
    screen.begin();
    ultrasonic.attach(TRIG_PIN, ECHO_PIN, MIN_DISTANCE, MAX_DISTANCE);
    screen.play(landscapeAnim, LOOP);
}

void loop()
{
    if (millis() - lastRead >= READ_INTERVAL)
    {
        lastRead = millis();
        distanceToFace = ultrasonic.getDistanceCM();

        // Determine which zone the face is in
        if (distanceToFace < NEAR_TO_MID)
        {
            currentZone = 1;  // NEAR — leaning in
        }
        else if (distanceToFace < MID_TO_FAR)
        {
            currentZone = 2;  // MIDDLE — normal viewing distance
        }
        else
        {
            currentZone = 3;  // FAR — standing back
        }

        // Print both values so we can verify the zones
        Serial.print("Distance: ");
        Serial.print(distanceToFace);
        Serial.print(" cm → Zone: ");
        Serial.println(currentZone);
    }

    // Animation plays independently — not connected to zones yet
    screen.update();
}
```

### What to notice

- **Two boundaries create three zones.** `NEAR_TO_MID` and `MID_TO_FAR` are all you need. Everything below the first is NEAR, everything above the second is FAR, everything between is MIDDLE.
- **The zone names describe the interaction.** `NEAR_TO_MID = 20` tells you that below 20 cm the face is considered "close." Compare that to `THRESHOLD_1 = 20` — same number, but no meaning.
- **The animation keeps playing.** Nothing on screen changes based on zone — we're just verifying the zone logic works before connecting it.
- **Tuning:** If the Serial Monitor shows the zone jumping erratically, widen the gap between thresholds or move them closer to where you actually position yourself.

---

## Step 5 — Connect Zones to Animation Speed

**Goal:** Each zone sets a different playback speed. The animation plays slow when your face is far, normal in the middle, and fast when close.

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
#include <EasyUltrasonic.h>
#include "landscape.h"

TinyScreen screen;
EasyUltrasonic ultrasonic;
Animation landscapeAnim = landscape;

// --- Pin configuration ---
int TRIG_PIN = A0;
int ECHO_PIN = A1;

// --- Sensor range ---
int MIN_DISTANCE = 2;
int MAX_DISTANCE = 80;

// --- Threshold boundaries (from YOUR observations) ---
int NEAR_TO_MID = 20;       // below = leaning in
int MID_TO_FAR = 50;        // above = standing back

// --- Speed per zone (as float multiplier) ---
// 1.0 = original speed, 2.0 = double, 0.5 = half
float NEAR_SPEED = 3.0;     // fast — leaning in, high energy
float MIDDLE_SPEED = 1.0;   // normal — default viewing pace
float FAR_SPEED = 0.3;      // slow — standing back, calm

// --- Timing ---
int READ_INTERVAL = 20;
unsigned long lastRead = 0;

// --- Values ---
float distanceToFace = 0;
int currentZone = 0;
float playbackSpeed = 1.0;

void setup()
{
    Serial.begin(9600);
    screen.begin();
    ultrasonic.attach(TRIG_PIN, ECHO_PIN, MIN_DISTANCE, MAX_DISTANCE);
    screen.play(landscapeAnim, LOOP);
}

void loop()
{
    if (millis() - lastRead >= READ_INTERVAL)
    {
        lastRead = millis();
        distanceToFace = ultrasonic.getDistanceCM();

        // ★ THE CONNECTION — zone determines the speed
        if (distanceToFace < NEAR_TO_MID)
        {
            currentZone = 1;
            playbackSpeed = NEAR_SPEED;
        }
        else if (distanceToFace < MID_TO_FAR)
        {
            currentZone = 2;
            playbackSpeed = MIDDLE_SPEED;
        }
        else
        {
            currentZone = 3;
            playbackSpeed = FAR_SPEED;
        }

        screen.setSpeed(playbackSpeed);

        Serial.print("Distance: ");
        Serial.print(distanceToFace);
        Serial.print(" cm → Zone ");
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
| **Zone boundaries** | `NEAR_TO_MID` / `MID_TO_FAR` | Shift where the speed changes happen |
| **Speed values** | `NEAR_SPEED` / `MIDDLE_SPEED` / `FAR_SPEED` | Try `0.0` for a full pause, `5.0` for frantic |
| **Number of zones** | Add more `else if` blocks + more speed values | More zones = more distinct speed levels |
| **Play mode** | `LOOP` in `screen.play()` | Try `BOOMERANG` for ping-pong playback |

### What's Next?

You now have the core pattern: **read → classify → apply**. Each zone sets a fixed property value — the animation jumps between speeds rather than sliding smoothly. From here you can control other animation properties per zone, or switch between entirely different animations. The examples below show how.

---

## Going Further

The examples below all start from the Step 5 code. Each one extends the threshold idea in a different direction. Comments highlight what's different.

---

### Example: Zone Controls Speed and Position

Each zone sets both a **speed** and a **position offset**. The animation speeds up and slides to the right as the viewer leans closer.

```cpp
#include "TinyFilmFestival.h"
#include <EasyUltrasonic.h>
#include "landscape.h"

TinyScreen screen;
EasyUltrasonic ultrasonic;
Animation landscapeAnim = landscape;

int TRIG_PIN = A0;
int ECHO_PIN = A1;

int MIN_DISTANCE = 2;
int MAX_DISTANCE = 80;

int NEAR_TO_MID = 20;
int MID_TO_FAR = 50;

// --- Speed per zone ---
float NEAR_SPEED = 3.0;
float MIDDLE_SPEED = 1.0;
float FAR_SPEED = 0.3;

// --- Position offset per zone ---
int NEAR_X = 3;          // shifted right when close
int MIDDLE_X = 0;        // centered
int FAR_X = -3;          // shifted left when far

int READ_INTERVAL = 20;
unsigned long lastRead = 0;

float distanceToFace = 0;
int currentZone = 0;
float playbackSpeed = 1.0;
int animationX = 0;

void setup()
{
    Serial.begin(9600);
    screen.begin();
    ultrasonic.attach(TRIG_PIN, ECHO_PIN, MIN_DISTANCE, MAX_DISTANCE);
    screen.play(landscapeAnim, LOOP);
}

void loop()
{
    if (millis() - lastRead >= READ_INTERVAL)
    {
        lastRead = millis();
        distanceToFace = ultrasonic.getDistanceCM();

        if (distanceToFace < NEAR_TO_MID)
        {
            currentZone = 1;
            playbackSpeed = NEAR_SPEED;
            animationX = NEAR_X;
        }
        else if (distanceToFace < MID_TO_FAR)
        {
            currentZone = 2;
            playbackSpeed = MIDDLE_SPEED;
            animationX = MIDDLE_X;
        }
        else
        {
            currentZone = 3;
            playbackSpeed = FAR_SPEED;
            animationX = FAR_X;
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
#include <EasyUltrasonic.h>
#include "landscape.h"

TinyScreen screen;
EasyUltrasonic ultrasonic;
Animation landscapeAnim = landscape;

int TRIG_PIN = A0;
int ECHO_PIN = A1;

int MIN_DISTANCE = 2;
int MAX_DISTANCE = 80;

int NEAR_TO_MID = 20;
int MID_TO_FAR = 50;

// --- Properties per zone ---
float NEAR_SPEED = 3.0;
float MIDDLE_SPEED = 1.0;
float FAR_SPEED = 0.3;

// NEAR zone inverts the display for a "danger" feel
// MIDDLE and FAR stay normal

int READ_INTERVAL = 20;
unsigned long lastRead = 0;

float distanceToFace = 0;
int currentZone = 0;

void setup()
{
    Serial.begin(9600);
    screen.begin();
    ultrasonic.attach(TRIG_PIN, ECHO_PIN, MIN_DISTANCE, MAX_DISTANCE);
    screen.play(landscapeAnim, LOOP);
}

void loop()
{
    if (millis() - lastRead >= READ_INTERVAL)
    {
        lastRead = millis();
        distanceToFace = ultrasonic.getDistanceCM();

        if (distanceToFace < NEAR_TO_MID)
        {
            currentZone = 1;
            screen.setSpeed(NEAR_SPEED);
            screen.setInvert(true);       // inverted — viewer is leaning in
        }
        else if (distanceToFace < MID_TO_FAR)
        {
            currentZone = 2;
            screen.setSpeed(MIDDLE_SPEED);
            screen.setInvert(false);      // normal
        }
        else
        {
            currentZone = 3;
            screen.setSpeed(FAR_SPEED);
            screen.setInvert(false);      // normal
        }
    }

    screen.update();
}
```

**What changed:** `screen.setInvert(true)` flips every pixel on the display. The NEAR zone inverts and speeds up — it feels urgent. MIDDLE and FAR stay normal. You could invert any zone, or combine invert with position changes for an even stronger effect.

---

### Example: Zone Switches Between Animations

Instead of changing properties of one animation, each zone **plays a different animation entirely**. The viewer leaning in or stepping back switches which animation is on screen.

> Download animations from the [exampleAnimations folder](https://github.com/DigitalFuturesOCADU/TinyFilmFestival/tree/main/exampleAnimations) or create your own with the [LED Matrix Editor](https://ledmatrix-editor.arduino.cc/). See the [Editor Guide](https://digitalfuturesocadu.github.io/TinyFilmFestival/docs/#editor-guide) for how to export `.h` files.

```cpp
#include "TinyFilmFestival.h"
#include <EasyUltrasonic.h>
#include "idle.h"
#include "fiz.h"
#include "landscape.h"

TinyScreen screen;
EasyUltrasonic ultrasonic;

// Each .h file's variable matches its filename
Animation idleAnim = idle;
Animation fizAnim = fiz;
Animation landscapeAnim = landscape;

int TRIG_PIN = A0;
int ECHO_PIN = A1;

int MIN_DISTANCE = 2;
int MAX_DISTANCE = 80;

int NEAR_TO_MID = 20;
int MID_TO_FAR = 50;

int READ_INTERVAL = 20;
unsigned long lastRead = 0;

float distanceToFace = 0;
int currentZone = 0;
int lastZone = 0;             // tracks previous zone to avoid restarting

void setup()
{
    Serial.begin(9600);
    screen.begin();
    ultrasonic.attach(TRIG_PIN, ECHO_PIN, MIN_DISTANCE, MAX_DISTANCE);
    screen.play(idleAnim, LOOP);   // start with FAR animation
    lastZone = 3;
}

void loop()
{
    if (millis() - lastRead >= READ_INTERVAL)
    {
        lastRead = millis();
        distanceToFace = ultrasonic.getDistanceCM();

        if (distanceToFace < NEAR_TO_MID)
        {
            currentZone = 1;
        }
        else if (distanceToFace < MID_TO_FAR)
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
            if (currentZone == 1) screen.play(landscapeAnim, LOOP);
            else if (currentZone == 2) screen.play(fizAnim, LOOP);
            else screen.play(idleAnim, LOOP);

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
#include <EasyUltrasonic.h>
#include "idle.h"
#include "fiz.h"
#include "landscape.h"

TinyScreen screen;
EasyUltrasonic ultrasonic;

Animation idleAnim = idle;
Animation fizAnim = fiz;
Animation landscapeAnim = landscape;

int TRIG_PIN = A0;
int ECHO_PIN = A1;

int MIN_DISTANCE = 2;
int MAX_DISTANCE = 80;

int NEAR_TO_MID = 20;
int MID_TO_FAR = 50;

// --- Speed per zone ---
float NEAR_SPEED = 2.5;     // fast playback for the NEAR animation
float MIDDLE_SPEED = 1.0;   // normal for MIDDLE
float FAR_SPEED = 0.5;      // slow for FAR

int READ_INTERVAL = 20;
unsigned long lastRead = 0;

float distanceToFace = 0;
int currentZone = 0;
int lastZone = 0;

void setup()
{
    Serial.begin(9600);
    screen.begin();
    ultrasonic.attach(TRIG_PIN, ECHO_PIN, MIN_DISTANCE, MAX_DISTANCE);
    screen.play(idleAnim, LOOP);
    screen.setSpeed(FAR_SPEED);
    lastZone = 3;
}

void loop()
{
    if (millis() - lastRead >= READ_INTERVAL)
    {
        lastRead = millis();
        distanceToFace = ultrasonic.getDistanceCM();

        if (distanceToFace < NEAR_TO_MID)
        {
            currentZone = 1;
        }
        else if (distanceToFace < MID_TO_FAR)
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
                screen.play(landscapeAnim, LOOP);
                screen.setSpeed(NEAR_SPEED);
            }
            else if (currentZone == 2)
            {
                screen.play(fizAnim, LOOP);
                screen.setSpeed(MIDDLE_SPEED);
            }
            else
            {
                screen.play(idleAnim, LOOP);
                screen.setSpeed(FAR_SPEED);
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
#include <EasyUltrasonic.h>
#include "landscape.h"          // background animation
#include "go.h"                 // foreground animation

TinyScreen screen;
EasyUltrasonic ultrasonic;

// Each .h file's variable matches its filename
Animation backgroundAnim = landscape;
Animation foregroundAnim = go;

int TRIG_PIN = A0;
int ECHO_PIN = A1;

int MIN_DISTANCE = 2;
int MAX_DISTANCE = 80;

int NEAR_TO_MID = 20;
int MID_TO_FAR = 50;

// --- Foreground speed per zone ---
float NEAR_FOREGROUND = 4.0;     // fast overlay when close
float MIDDLE_FOREGROUND = 1.0;   // normal
float FAR_FOREGROUND = 0.2;      // barely moving when far

int READ_INTERVAL = 20;
unsigned long lastRead = 0;

float distanceToFace = 0;
int currentZone = 0;
float foregroundSpeed = 1.0;

void setup()
{
    Serial.begin(9600);
    screen.begin();
    ultrasonic.attach(TRIG_PIN, ECHO_PIN, MIN_DISTANCE, MAX_DISTANCE);

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
        distanceToFace = ultrasonic.getDistanceCM();

        if (distanceToFace < NEAR_TO_MID)
        {
            currentZone = 1;
            foregroundSpeed = NEAR_FOREGROUND;
        }
        else if (distanceToFace < MID_TO_FAR)
        {
            currentZone = 2;
            foregroundSpeed = MIDDLE_FOREGROUND;
        }
        else
        {
            currentZone = 3;
            foregroundSpeed = FAR_FOREGROUND;
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

The animation oscillates back and forth using `oscillateInt()`, but each zone sets a **different oscillation speed**. NEAR jitters fast, MIDDLE drifts gently, FAR barely moves.

```cpp
#include "TinyFilmFestival.h"
#include <EasyUltrasonic.h>
#include "landscape.h"

TinyScreen screen;
EasyUltrasonic ultrasonic;
Animation landscapeAnim = landscape;

int TRIG_PIN = A0;
int ECHO_PIN = A1;

int MIN_DISTANCE = 2;
int MAX_DISTANCE = 80;

int NEAR_TO_MID = 20;
int MID_TO_FAR = 50;

// --- Oscillation range ---
int LEFT_LIMIT = -3;
int RIGHT_LIMIT = 3;

// --- Oscillation speed per zone (ms per cycle) ---
int NEAR_SLIDE = 400;       // fast jitter
int MIDDLE_SLIDE = 1500;    // gentle drift
int FAR_SLIDE = 4000;       // barely moving

int READ_INTERVAL = 20;
unsigned long lastRead = 0;

float distanceToFace = 0;
int currentZone = 0;
int slideSpeed = 1500;

void setup()
{
    Serial.begin(9600);
    screen.begin();
    ultrasonic.attach(TRIG_PIN, ECHO_PIN, MIN_DISTANCE, MAX_DISTANCE);
    screen.play(landscapeAnim, LOOP);
}

void loop()
{
    if (millis() - lastRead >= READ_INTERVAL)
    {
        lastRead = millis();
        distanceToFace = ultrasonic.getDistanceCM();

        if (distanceToFace < NEAR_TO_MID)
        {
            currentZone = 1;
            slideSpeed = NEAR_SLIDE;
        }
        else if (distanceToFace < MID_TO_FAR)
        {
            currentZone = 2;
            slideSpeed = MIDDLE_SLIDE;
        }
        else
        {
            currentZone = 3;
            slideSpeed = FAR_SLIDE;
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

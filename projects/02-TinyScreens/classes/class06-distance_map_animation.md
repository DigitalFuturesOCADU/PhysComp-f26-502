# Distance Sensor → Animation: Step by Step

[← Back to Class 06](class06-Feb13.md)

This walkthrough takes you from a blank animation to a sensor-controlled animation in **four steps**. Each step adds one idea and builds directly on the previous code. Don't skip ahead — the point is to understand what each piece does before the next one appears.

If you've already done the [Distance Sensor → Screen](class06-distance_map_canvas.md) walkthrough, this follows the same incremental structure — but instead of drawing shapes on a canvas, you're controlling a **pre-made animation**.

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

1. Click the **Export H** button to download your animation as a `.h` file.
2. **Name the file with no spaces** — for example `myAnimation.h` or `walkCycle.h`.
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

This is the same sensor setup from [Distance to Screen — Step 1](class06-distance_map_canvas.md#step-1--read-the-sensor). The animation keeps playing independently while the sensor prints values in the background.

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
//   can map it to anything. Move your hand and write down
//   the closest and farthest reliable values you see.
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
float distanceCM = 0;

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
        distanceCM = ultrasonic.getDistanceCM();

        Serial.print("Distance (cm): ");
        Serial.println(distanceCM);
    }

    // --- Animation keeps playing independently ---
    screen.update();
}
```

### What to do with this

1. Upload and open the **Serial Monitor** (9600 baud).
2. The animation plays on the matrix while distance values scroll in the monitor.
3. Move your hand toward and away from the sensor.
4. **Write down two numbers:**
   - The **closest** reliable value — this is your sensor minimum
   - The **farthest** reliable value — this is your sensor maximum

These become `CLOSEST_CM` and `FARTHEST_CM` in the next step.

---

## Step 4 — Connect Sensor to Animation Speed

**Goal:** Use `map()` to convert distance into a speed value, then pass it to `screen.setSpeed()` so the animation plays faster when your hand is closer.

`setSpeed()` accepts either **milliseconds per frame** (integer) or a **multiplier** (float, where `1.0` = original speed, `2.0` = double speed, `0.5` = half speed). We'll use the multiplier approach so the numbers are easier to reason about.

```cpp
// ============================================================
// Step 4 — Distance controls animation speed
// ============================================================
// What changed:
//   - Variables renamed to describe the interaction:
//     distanceToHand, CLOSEST_CM, FARTHEST_CM,
//     SLOWEST_SPEED, FASTEST_SPEED, playbackSpeed
//   - map() converts distance → speed percentage
//   - Division by 100.0 converts to the float multiplier
//     that setSpeed() expects
//   - Closer hand = faster animation
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

// --- Sensor range (from YOUR Serial Monitor observations) ---
int CLOSEST_CM = 5;       // update with your real minimum
int FARTHEST_CM = 80;     // update with your real maximum

// --- Speed range (as percentages of original speed) ---
// We map to integers (50–300) then divide by 100.0 to get a float.
// This avoids dealing with float mapping directly.
int SLOWEST_SPEED = 50;   // 50% — half speed when hand is far
int FASTEST_SPEED = 300;  // 300% — triple speed when hand is close

// --- Timing ---
int READ_INTERVAL = 20;
unsigned long lastRead = 0;

// --- Values ---
float distanceToHand = 0;     // how far the hand is (cm)
int speedPercent = 100;        // mapped speed as a percentage
float playbackSpeed = 1.0;    // what we pass to setSpeed()

void setup()
{
    Serial.begin(9600);
    screen.begin();
    ultrasonic.attach(TRIG_PIN, ECHO_PIN, CLOSEST_CM, FARTHEST_CM);
    screen.play(landscapeAnim, LOOP);
}

void loop()
{
    if (millis() - lastRead >= READ_INTERVAL)
    {
        lastRead = millis();

        // Read how far the hand is
        distanceToHand = ultrasonic.getDistanceCM();

        // ★ THE CONNECTION — distance drives the animation speed
        // Closer hand = higher speed percentage = faster playback
        speedPercent = map(distanceToHand, CLOSEST_CM, FARTHEST_CM, FASTEST_SPEED, SLOWEST_SPEED);
        speedPercent = constrain(speedPercent, SLOWEST_SPEED, FASTEST_SPEED);

        // Convert percentage to the float multiplier setSpeed() expects
        // 50 → 0.5, 100 → 1.0, 300 → 3.0
        playbackSpeed = speedPercent / 100.0;

        screen.setSpeed(playbackSpeed);

        // Debug printing
        Serial.print("Distance: ");
        Serial.print(distanceToHand);
        Serial.print(" cm → Speed: ");
        Serial.print(playbackSpeed);
        Serial.println("x");
    }

    screen.update();
}
```

### Tuning the Interaction

| What to change | Where | Effect |
|---|---|---|
| **Sensor range** | `CLOSEST_CM` / `FARTHEST_CM` | Narrower range = more sensitive. Use the real values from Step 3. |
| **Speed range** | `SLOWEST_SPEED` / `FASTEST_SPEED` | `50` = half speed, `100` = normal, `200` = double, `300` = triple. Try `25` for very slow. |
| **Map direction** | Swap `FASTEST_SPEED` and `SLOWEST_SPEED` in `map()` | Reverses the interaction — closer hand = slower instead of faster. |
| **Read speed** | `READ_INTERVAL` | Lower = more responsive speed changes. Higher = smoother but slower to react. |
| **Play mode** | `LOOP` in `screen.play()` | Try `BOOMERANG` for ping-pong playback, or `ONCE` to play a single pass and stop. |

### What's Next?

You now have the core pattern: **read → translate → apply**. From here you can control other animation properties with the same sensor. The examples below show how.

---

## Going Further

The examples below all start from the Step 4 code. Each one changes what the sensor controls — position, layers, inversion — using the same **read → translate → apply** pattern. Comments highlight what's different.

---

### Example: Distance Controls Animation Position

Instead of controlling **speed**, the sensor controls **where the animation appears** on the matrix. The animation slides left and right as your hand moves.

`setPosition(x, y)` shifts the animation by an offset — pixels that move beyond the 12×8 edges are automatically clipped.

```cpp
#include "TinyFilmFestival.h"
#include <EasyUltrasonic.h>
#include "landscape.h"          // ← replace with YOUR .h filename

TinyScreen screen;
EasyUltrasonic ultrasonic;
Animation landscapeAnim = landscape;

int TRIG_PIN = A0;
int ECHO_PIN = A1;

// --- Sensor range ---
int CLOSEST_CM = 5;
int FARTHEST_CM = 80;

// --- Position range ---
// Named for what they mean on screen
int LEFT_OFFSET = -4;     // shift animation left (partially off-screen)
int RIGHT_OFFSET = 4;     // shift animation right (partially off-screen)

int READ_INTERVAL = 20;
unsigned long lastRead = 0;

float distanceToHand = 0;     // how far the hand is
int animationX = 0;            // horizontal offset for the animation

void setup()
{
    Serial.begin(9600);
    screen.begin();
    ultrasonic.attach(TRIG_PIN, ECHO_PIN, CLOSEST_CM, FARTHEST_CM);
    screen.play(landscapeAnim, LOOP);
}

void loop()
{
    if (millis() - lastRead >= READ_INTERVAL)
    {
        lastRead = millis();
        distanceToHand = ultrasonic.getDistanceCM();

        // Closer hand = animation shifts right, farther = shifts left
        animationX = map(distanceToHand, CLOSEST_CM, FARTHEST_CM, RIGHT_OFFSET, LEFT_OFFSET);
        animationX = constrain(animationX, LEFT_OFFSET, RIGHT_OFFSET);

        // Apply the position offset
        screen.setPosition(animationX, 0);

        Serial.print("Distance: ");
        Serial.print(distanceToHand);
        Serial.print(" cm → X offset: ");
        Serial.println(animationX);
    }

    screen.update();
}
```

**What changed:** `map()` now feeds `animationX` and passes it to `screen.setPosition()` instead of `screen.setSpeed()`. The animation still plays at its original speed — only its position moves. Negative offsets shift left (clipping at the edge), positive shift right.

---

### Example: Distance Controls Vertical Position

Same idea, but the animation moves **up and down** instead of left and right.

```cpp
#include "TinyFilmFestival.h"
#include <EasyUltrasonic.h>
#include "landscape.h"

TinyScreen screen;
EasyUltrasonic ultrasonic;
Animation landscapeAnim = landscape;

int TRIG_PIN = A0;
int ECHO_PIN = A1;

int CLOSEST_CM = 5;
int FARTHEST_CM = 80;

// --- Vertical position range ---
int TOP_OFFSET = -3;      // push animation up (partially off-screen)
int BOTTOM_OFFSET = 3;    // push animation down (partially off-screen)

int READ_INTERVAL = 20;
unsigned long lastRead = 0;

float distanceToHand = 0;
int animationY = 0;            // vertical offset for the animation

void setup()
{
    Serial.begin(9600);
    screen.begin();
    ultrasonic.attach(TRIG_PIN, ECHO_PIN, CLOSEST_CM, FARTHEST_CM);
    screen.play(landscapeAnim, LOOP);
}

void loop()
{
    if (millis() - lastRead >= READ_INTERVAL)
    {
        lastRead = millis();
        distanceToHand = ultrasonic.getDistanceCM();

        // Closer hand = animation moves up, farther = moves down
        animationY = map(distanceToHand, CLOSEST_CM, FARTHEST_CM, TOP_OFFSET, BOTTOM_OFFSET);
        animationY = constrain(animationY, TOP_OFFSET, BOTTOM_OFFSET);

        screen.setPosition(0, animationY);
    }

    screen.update();
}
```

**What changed:** The position offset is on the **Y axis** instead of X. `screen.setPosition(0, animationY)` keeps horizontal position at 0 and moves the animation vertically. Closer hand = up (lower Y), farther = down (higher Y).

---

### Example: Distance Controls Speed AND Position

The sensor drives **two properties at once** — the animation speeds up and slides right as your hand gets closer.

```cpp
#include "TinyFilmFestival.h"
#include <EasyUltrasonic.h>
#include "landscape.h"

TinyScreen screen;
EasyUltrasonic ultrasonic;
Animation landscapeAnim = landscape;

int TRIG_PIN = A0;
int ECHO_PIN = A1;

int CLOSEST_CM = 5;
int FARTHEST_CM = 80;

// --- Speed range (as percentage) ---
int SLOWEST_SPEED = 50;
int FASTEST_SPEED = 300;

// --- Position range ---
int LEFT_OFFSET = -4;
int RIGHT_OFFSET = 4;

int READ_INTERVAL = 20;
unsigned long lastRead = 0;

float distanceToHand = 0;
int speedPercent = 100;
float playbackSpeed = 1.0;
int animationX = 0;

void setup()
{
    Serial.begin(9600);
    screen.begin();
    ultrasonic.attach(TRIG_PIN, ECHO_PIN, CLOSEST_CM, FARTHEST_CM);
    screen.play(landscapeAnim, LOOP);
}

void loop()
{
    if (millis() - lastRead >= READ_INTERVAL)
    {
        lastRead = millis();
        distanceToHand = ultrasonic.getDistanceCM();

        // Speed: closer = faster
        speedPercent = map(distanceToHand, CLOSEST_CM, FARTHEST_CM, FASTEST_SPEED, SLOWEST_SPEED);
        speedPercent = constrain(speedPercent, SLOWEST_SPEED, FASTEST_SPEED);
        playbackSpeed = speedPercent / 100.0;
        screen.setSpeed(playbackSpeed);

        // Position: closer = further right
        animationX = map(distanceToHand, CLOSEST_CM, FARTHEST_CM, RIGHT_OFFSET, LEFT_OFFSET);
        animationX = constrain(animationX, LEFT_OFFSET, RIGHT_OFFSET);
        screen.setPosition(animationX, 0);
    }

    screen.update();
}
```

**What changed:** Two `map()` calls from the same sensor — one for speed, one for position. Same pattern as the [Two Shapes example](class06-distance_map_canvas.md#example-two-shapes-from-one-sensor) in the canvas walkthrough. You can map one sensor to as many animation parameters as you want.

---

### Example: Two Layered Animations at Different Speeds

Stack two animations on top of each other. The sensor controls the **foreground speed** while the background plays at a fixed rate. Lit pixels on the foreground layer appear on top of the background.

```cpp
#include "TinyFilmFestival.h"
#include <EasyUltrasonic.h>
#include "landscape.h"          // background animation
#include "go.h"                 // foreground animation — use a different .h file

TinyScreen screen;
EasyUltrasonic ultrasonic;

// Each .h file's variable matches its filename:
// landscape.h → landscape, go.h → go
Animation backgroundAnim = landscape;
Animation foregroundAnim = go;

int TRIG_PIN = A0;
int ECHO_PIN = A1;

int CLOSEST_CM = 5;
int FARTHEST_CM = 80;

// --- Foreground speed range ---
int SLOWEST_FOREGROUND = 30;    // 30% speed when far
int FASTEST_FOREGROUND = 400;   // 4x speed when close

int READ_INTERVAL = 20;
unsigned long lastRead = 0;

float distanceToHand = 0;
int speedPercent = 100;
float foregroundSpeed = 1.0;

void setup()
{
    Serial.begin(9600);
    screen.begin();
    ultrasonic.attach(TRIG_PIN, ECHO_PIN, CLOSEST_CM, FARTHEST_CM);

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
        distanceToHand = ultrasonic.getDistanceCM();

        // Only the foreground speed changes — background stays constant
        speedPercent = map(distanceToHand, CLOSEST_CM, FARTHEST_CM, FASTEST_FOREGROUND, SLOWEST_FOREGROUND);
        speedPercent = constrain(speedPercent, SLOWEST_FOREGROUND, FASTEST_FOREGROUND);
        foregroundSpeed = speedPercent / 100.0;

        screen.setSpeedOnLayer(1, foregroundSpeed);
    }

    screen.update();
}
```

**What changed:** `screen.addLayer()` creates a second animation layer. The background plays on layer 0 at its original speed. The foreground plays on layer 1 with its speed controlled by the sensor via `screen.setSpeedOnLayer(1, ...)`. Lit pixels on the foreground layer are drawn on top of the background. Each `.h` file exports a variable matching its filename (`landscape` and `go`), so there's no naming conflict.

---

### Example: Distance Inverts the Display

At a threshold distance, the entire display **inverts** — all lit pixels turn off and all off pixels turn on. This creates a dramatic visual flip with a single API call.

```cpp
#include "TinyFilmFestival.h"
#include <EasyUltrasonic.h>
#include "landscape.h"

TinyScreen screen;
EasyUltrasonic ultrasonic;
Animation landscapeAnim = landscape;

int TRIG_PIN = A0;
int ECHO_PIN = A1;

int CLOSEST_CM = 5;
int FARTHEST_CM = 80;

// --- Threshold ---
// When the hand crosses this distance, the display inverts
int INVERT_THRESHOLD = 30;    // cm — halfway through the usable range

int READ_INTERVAL = 20;
unsigned long lastRead = 0;

float distanceToHand = 0;

void setup()
{
    Serial.begin(9600);
    screen.begin();
    ultrasonic.attach(TRIG_PIN, ECHO_PIN, CLOSEST_CM, FARTHEST_CM);
    screen.play(landscapeAnim, LOOP);
}

void loop()
{
    if (millis() - lastRead >= READ_INTERVAL)
    {
        lastRead = millis();
        distanceToHand = ultrasonic.getDistanceCM();

        // Invert when hand is closer than the threshold
        if (distanceToHand < INVERT_THRESHOLD)
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

The animation oscillates back and forth automatically using `oscillateInt()`, but the **speed of the oscillation** is controlled by distance. This combines the animation's own playback with an additional layer of motion.

```cpp
#include "TinyFilmFestival.h"
#include <EasyUltrasonic.h>
#include "landscape.h"

TinyScreen screen;
EasyUltrasonic ultrasonic;
Animation landscapeAnim = landscape;

int TRIG_PIN = A0;
int ECHO_PIN = A1;

int CLOSEST_CM = 5;
int FARTHEST_CM = 80;

// --- Oscillation range ---
int LEFT_LIMIT = -3;       // furthest left the animation slides
int RIGHT_LIMIT = 3;       // furthest right

// --- Oscillation speed range ---
int FASTEST_SLIDE = 500;    // ms — fast slide when hand is close
int SLOWEST_SLIDE = 4000;   // ms — slow drift when hand is far

int READ_INTERVAL = 20;
unsigned long lastRead = 0;

float distanceToHand = 0;
int slideSpeed = 2000;         // ms — period of the oscillation

void setup()
{
    Serial.begin(9600);
    screen.begin();
    ultrasonic.attach(TRIG_PIN, ECHO_PIN, CLOSEST_CM, FARTHEST_CM);
    screen.play(landscapeAnim, LOOP);
}

void loop()
{
    if (millis() - lastRead >= READ_INTERVAL)
    {
        lastRead = millis();
        distanceToHand = ultrasonic.getDistanceCM();

        // Map distance to oscillation period
        slideSpeed = map(distanceToHand, CLOSEST_CM, FARTHEST_CM, FASTEST_SLIDE, SLOWEST_SLIDE);
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

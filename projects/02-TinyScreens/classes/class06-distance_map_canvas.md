# Distance Sensor → Screen: Step by Step

[← Back to Class 06](class06-Feb13.md)

This walkthrough takes you from a bare sensor reading to a working on-screen interaction in **four steps**. Each step adds one idea and builds directly on the previous code. Don't skip ahead — the point is to understand what each piece does before the next one appears.

---

## Step 1 — Read the Sensor

**Goal:** Confirm the sensor is wired correctly and see the raw values it produces.

Wire the HC-SR04 as shown in the [Class 06 wiring diagram](class06-Feb13.md#distance-sensor--how-does-it-work). Then upload this sketch and open **Tools → Serial Monitor** at 9600 baud.

```cpp
// ============================================================
// Step 1 — Read the distance sensor and print to Serial Monitor
// ============================================================
// What this does:
//   Reads the HC-SR04 distance sensor every 20 ms
//   and prints the value (in cm) to the Serial Monitor.
//
// Why this matters:
//   You need to SEE the real numbers your sensor produces
//   before you can do anything useful with them.
// ============================================================

#include <EasyUltrasonic.h>

EasyUltrasonic ultrasonic;

// --- Pin configuration ---
// These must match how you wired the sensor
int TRIG_PIN = A0;   // Arduino pin → HC-SR04 TRIG
int ECHO_PIN = A1;   // Arduino pin → HC-SR04 ECHO

// --- Sensor range ---
int MIN_DISTANCE = 2;    // cm — closest reliable reading
int MAX_DISTANCE = 100;  // cm — farthest we care about

// --- Timing ---
// We read the sensor on a timer instead of using delay()
// so the rest of the code can keep running smoothly.
int READ_INTERVAL = 20;       // ms between sensor reads
unsigned long lastRead = 0;   // tracks when we last read

// --- Sensor value ---
float distanceCM = 0;         // holds the most recent reading

void setup()
{
    Serial.begin(9600);
    ultrasonic.attach(TRIG_PIN, ECHO_PIN, MIN_DISTANCE, MAX_DISTANCE);
}

void loop()
{
    // Only read the sensor when enough time has passed
    if (millis() - lastRead >= READ_INTERVAL)
    {
        lastRead = millis();

        // Read the sensor — result is in centimeters
        distanceCM = ultrasonic.getDistanceCM();

        // Print so we can see what the sensor is actually returning
        Serial.print("Distance (cm): ");
        Serial.println(distanceCM);
    }
}
```

### What to do with this

1. Upload the code and open the **Serial Monitor** (Tools → Serial Monitor, 9600 baud)
2. Move your hand slowly toward and away from the sensor
3. **Write down two numbers:**
   - The **closest** value you see reliably (hand very close) — this is your **sensor minimum**
   - The **farthest** value before readings get jumpy — this is your **sensor maximum**

You'll use these numbers in the next step. Every sensor and setup is slightly different, so **measure yours — don't guess**.

---

## Step 2 — Map the Value

**Goal:** Convert the raw sensor range into a useful output range and verify it with Serial Monitor.

In this step we add `map()` to translate the sensor's centimeter values into a new range: **1 to 12**. These will eventually become the diameter of a circle, but for now we just print both values so we can confirm the mapping works.

We also start **naming variables by what they represent** in the interaction — not generic labels like `val` or `mappedValue`, but names that describe the data: `distanceToHand`, `CLOSEST_CM`, `LARGEST_DIAMETER`. See [Input Language](class06-CodePatterns.md#input-language) for why this matters.

```cpp
// ============================================================
// Step 2 — Map the distance to an output range
// ============================================================
// What's new:
//   - Variable names describe their purpose:
//     distanceToHand, CLOSEST_CM, LARGEST_DIAMETER, etc.
//   - CLOSEST_CM / FARTHEST_CM — your real measured range
//   - SMALLEST_DIAMETER / LARGEST_DIAMETER — the output range
//   - mappedSize — the result of the mapping
//   - map() converts one range to another
//   - constrain() clamps the result so it stays in bounds
//
// What to watch:
//   The Serial Monitor now prints BOTH values side by side
//   so you can see the mapping in action.
// ============================================================

#include <EasyUltrasonic.h>

EasyUltrasonic ultrasonic;

// --- Pin configuration ---
int TRIG_PIN = A0;
int ECHO_PIN = A1;

// --- Sensor range (from YOUR Serial Monitor observations) ---
// Replace these with the real min/max you wrote down in Step 1
// Name them for what they mean, not just "min" and "max"
int CLOSEST_CM = 2;      // cm — closest reliable reading
int FARTHEST_CM = 100;   // cm — farthest reliable reading

// --- Output range ---
// These define what we want to map the sensor values TO.
// We're using circle diameter, so 1 = tiny, 12 = fills the screen.
int SMALLEST_DIAMETER = 1;
int LARGEST_DIAMETER = 12;

// --- Timing ---
int READ_INTERVAL = 20;
unsigned long lastRead = 0;

// --- Values ---
// Name variables by what they represent in your interaction
float distanceToHand = 0;    // how far away the hand is (cm)
int mappedSize = 0;           // what diameter the map() produces

void setup()
{
    Serial.begin(9600);
    ultrasonic.attach(TRIG_PIN, ECHO_PIN, CLOSEST_CM, FARTHEST_CM);
}

void loop()
{
    if (millis() - lastRead >= READ_INTERVAL)
    {
        lastRead = millis();

        // READ — get the distance to the hand
        distanceToHand = ultrasonic.getDistanceCM();

        // TRANSLATE — convert distance → circle diameter
        // Closer hand = bigger circle (note the reversed output range)
        // map(value, fromLow, fromHigh, toLow, toHigh)
        mappedSize = map(distanceToHand, CLOSEST_CM, FARTHEST_CM, LARGEST_DIAMETER, SMALLEST_DIAMETER);

        // constrain() keeps the result within bounds
        // in case the sensor returns a value outside our expected range
        mappedSize = constrain(mappedSize, SMALLEST_DIAMETER, LARGEST_DIAMETER);

        // Print both values so we can verify the mapping
        Serial.print("Hand distance: ");
        Serial.print(distanceToHand);
        Serial.print(" cm → Circle diameter: ");
        Serial.println(mappedSize);
    }
}
```

### What to notice

- **`map()` direction matters.** We map `CLOSEST_CM → LARGEST_DIAMETER` and `FARTHEST_CM → SMALLEST_DIAMETER`. This means **closer = bigger circle**. If you want the opposite, swap the diameter values in the `map()` call.
- **`constrain()`** is a safety net. If the sensor returns a value outside your expected range, `constrain()` clips it so `mappedSize` never goes below 1 or above 12.
- **Tuning:** If the numbers in the Serial Monitor don't spread across the full 1–12 range when you move your hand, adjust `CLOSEST_CM` and `FARTHEST_CM` to better match what you actually see.
- **Variable names tell the story.** Reading `distanceToHand` and `LARGEST_DIAMETER` in the code, you immediately know what the data represents. Compare that to `distanceCM` and `OUTPUT_MAX` — those describe the *format* of the data but not its *meaning*.

---

## Step 3 — Draw a Static Circle

**Goal:** Get the TinyFilmFestival library drawing a circle on the LED matrix with a fixed diameter.

Before connecting the sensor, we add the screen and draw a circle at a **fixed size**. This follows the [From Static to Dynamic](class06-CodePatterns.md#from-static-to-dynamic) pattern: start with a hard-coded value, then replace it with sensor data in the next step.

```cpp
// ============================================================
// Step 3 — Draw a circle with a fixed diameter
// ============================================================
// What's new:
//   - TinyFilmFestival library added
//   - screen object created and initialized
//   - circleDiameter — this is the PARAMETER we will
//     connect to the sensor in the next step
//   - beginDraw / endDraw block draws the circle each frame
//
// The sensor code is still here (still printing to Serial)
// but the screen is NOT connected to it yet.
// ============================================================

#include "TinyFilmFestival.h"
#include <EasyUltrasonic.h>

TinyScreen screen;
EasyUltrasonic ultrasonic;

// --- Pin configuration ---
int TRIG_PIN = A0;
int ECHO_PIN = A1;

// --- Sensor range ---
int CLOSEST_CM = 2;
int FARTHEST_CM = 100;

// --- Output range ---
int SMALLEST_DIAMETER = 1;
int LARGEST_DIAMETER = 12;

// --- Timing ---
int READ_INTERVAL = 20;
unsigned long lastRead = 0;

// --- Values ---
float distanceToHand = 0;    // how far away the hand is (cm)
int mappedSize = 0;           // what diameter the map() produces

// --- Screen parameter ---
// This is the value that will eventually be driven by the sensor.
// For now it's fixed so we can confirm the circle draws correctly.
int circleDiameter = 6;       // fixed — we'll connect it in Step 4

void setup()
{
    Serial.begin(9600);
    screen.begin();
    ultrasonic.attach(TRIG_PIN, ECHO_PIN, CLOSEST_CM, FARTHEST_CM);
}

void loop()
{
    // --- Sensor reading (still running, still printing) ---
    if (millis() - lastRead >= READ_INTERVAL)
    {
        lastRead = millis();

        distanceToHand = ultrasonic.getDistanceCM();
        mappedSize = map(distanceToHand, CLOSEST_CM, FARTHEST_CM, LARGEST_DIAMETER, SMALLEST_DIAMETER);
        mappedSize = constrain(mappedSize, SMALLEST_DIAMETER, LARGEST_DIAMETER);

        Serial.print("Hand distance: ");
        Serial.print(distanceToHand);
        Serial.print(" cm → Mapped diameter: ");
        Serial.println(mappedSize);
    }

    // --- Draw the circle ---
    // beginDraw / endDraw wraps all drawing commands for one frame
    screen.beginDraw();
    screen.background(OFF);     // clear the screen
    screen.stroke(ON);          // outline ON (needed for circle)
    screen.fill(ON);            // fill the circle solid
    screen.circle(6, 4, circleDiameter);  // center of 12×8 grid, fixed diameter
    screen.endDraw();
}
```

### What to notice

- `circleDiameter` is a variable, not a raw number inside `screen.circle()`. Its name tells you exactly what it controls. This is the key setup for Step 4 — we've already isolated the parameter.
- The sensor is still reading and printing, but **nothing on screen changes yet**. The circle stays the same size no matter what the sensor reads.
- `screen.circle(6, 4, circleDiameter)` draws at roughly the center of the 12×8 LED matrix. The `6` is the x position (horizontal center) and `4` is the y position (vertical center).

---

## Step 4 — Connect Sensor to Screen

**Goal:** Replace the fixed diameter with the mapped sensor value so the circle responds to your hand.

This is the payoff. One line changes: `circleDiameter` is now computed from `map()` instead of being fixed at 6. The circle on screen grows and shrinks as you move your hand closer and farther from the sensor.

```cpp
// ============================================================
// Step 4 — Sensor controls the circle diameter
// ============================================================
// What changed:
//   circleDiameter is now computed from map() instead of
//   being fixed at 6. The structure from Steps 1–3 did all
//   the heavy lifting. This step just connects the pieces.
// ============================================================

#include "TinyFilmFestival.h"
#include <EasyUltrasonic.h>

TinyScreen screen;
EasyUltrasonic ultrasonic;

// --- Pin configuration ---
int TRIG_PIN = A0;
int ECHO_PIN = A1;

// --- Sensor range (from YOUR Serial Monitor observations) ---
int CLOSEST_CM = 2;      // cm — update with your real minimum
int FARTHEST_CM = 100;   // cm — update with your real maximum

// --- Output range ---
int SMALLEST_DIAMETER = 1;      // smallest circle
int LARGEST_DIAMETER = 12;      // largest circle

// --- Timing ---
int READ_INTERVAL = 20;
unsigned long lastRead = 0;

// --- Values ---
float distanceToHand = 0;       // how far away the hand is (cm)
int circleDiameter = 0;          // now driven by the sensor

void setup()
{
    Serial.begin(9600);
    screen.begin();
    ultrasonic.attach(TRIG_PIN, ECHO_PIN, CLOSEST_CM, FARTHEST_CM);
}

void loop()
{
    // --- READ and TRANSLATE ---
    if (millis() - lastRead >= READ_INTERVAL)
    {
        lastRead = millis();

        // Read the distance to the hand
        distanceToHand = ultrasonic.getDistanceCM();

        // ★ THE CONNECTION — sensor drives the circle diameter directly
        // Closer hand = bigger circle
        circleDiameter = map(distanceToHand, CLOSEST_CM, FARTHEST_CM, LARGEST_DIAMETER, SMALLEST_DIAMETER);
        circleDiameter = constrain(circleDiameter, SMALLEST_DIAMETER, LARGEST_DIAMETER);

        // Still printing so you can debug
        Serial.print("Hand distance: ");
        Serial.print(distanceToHand);
        Serial.print(" cm → Circle diameter: ");
        Serial.println(circleDiameter);
    }

    // --- DRAW ---
    screen.beginDraw();
    screen.background(OFF);
    screen.stroke(ON);
    screen.fill(ON);
    screen.circle(6, 4, circleDiameter);   // diameter changes with distance!
    screen.endDraw();
}
```

### Tuning the Interaction

The code works, but the *feel* of the interaction depends on the numbers. Here's what to adjust:

| What to change | Where | Effect |
|---|---|---|
| **Sensor range** | `CLOSEST_CM` / `FARTHEST_CM` | Narrower range = more sensitive in a smaller zone. If you only care about 5–40 cm, set those as your min/max. |
| **Output range** | `SMALLEST_DIAMETER` / `LARGEST_DIAMETER` | Change the smallest and largest circle sizes. Try `3` to `10` for a subtler effect. |
| **Map direction** | Swap `LARGEST_DIAMETER` and `SMALLEST_DIAMETER` in `map()` | Reverses the interaction — closer = smaller instead of bigger. |
| **Circle position** | `screen.circle(6, 4, ...)` | Move the circle to a different spot on the matrix. `(0, 0)` = top-left, `(11, 7)` = bottom-right. |
| **Read speed** | `READ_INTERVAL` | Lower = more responsive but potentially jittery. Higher = smoother but slower to react. |

### What's Next?

You now have the core pattern: **read → translate → draw**. Every sensor example in [Class 06](class06-Feb13.md) follows this same structure. From here you can:

- Control **animation speed** instead of circle size — see [Distance Controls Animation Speed](class06-Feb13.md#example-distance-controls-animation-speed)
- **Switch between animations** at a threshold — see [Distance Threshold](class06-Feb13.md#example-distance-threshold--switch-animations)
- Control **position** instead of size — see [Distance Mapped to Canvas](class06-Feb13.md#example-distance-mapped-to-canvas)
- Try the same pattern with a **pressure sensor** — see [Pressure Sensor](class06-Feb13.md#pressure-sensor--custom-fsr)

---

## Going Further

The examples below all start from the Step 4 code. Each one changes or adds a small amount to show a different way to use the same sensor data. Comments in the code highlight what's different.

---

### Example: Two Shapes from One Sensor

The sensor controls **two things at once** — a circle's diameter and a rectangle's width. Both use `distanceToHand`, but each maps it to a different output range with a name that describes what it controls.

```cpp
#include "TinyFilmFestival.h"
#include <EasyUltrasonic.h>

TinyScreen screen;
EasyUltrasonic ultrasonic;

int TRIG_PIN = A0;
int ECHO_PIN = A1;

// --- Sensor range ---
int CLOSEST_CM = 2;
int FARTHEST_CM = 100;

// --- Output ranges — named for what each shape needs ---
int SMALLEST_CIRCLE = 1;
int LARGEST_CIRCLE = 6;
int NARROWEST_BAR = 0;
int WIDEST_BAR = 12;

int READ_INTERVAL = 20;
unsigned long lastRead = 0;

float distanceToHand = 0;       // the input
int circleDiameter = 0;          // output: how big the circle is
int barWidth = 0;                // output: how wide the rectangle is

void setup()
{
    Serial.begin(9600);
    screen.begin();
    ultrasonic.attach(TRIG_PIN, ECHO_PIN, CLOSEST_CM, FARTHEST_CM);
}

void loop()
{
    if (millis() - lastRead >= READ_INTERVAL)
    {
        lastRead = millis();
        distanceToHand = ultrasonic.getDistanceCM();

        // Same sensor, two different map() calls — each named for its purpose
        circleDiameter = map(distanceToHand, CLOSEST_CM, FARTHEST_CM, LARGEST_CIRCLE, SMALLEST_CIRCLE);
        circleDiameter = constrain(circleDiameter, SMALLEST_CIRCLE, LARGEST_CIRCLE);

        barWidth = map(distanceToHand, CLOSEST_CM, FARTHEST_CM, WIDEST_BAR, NARROWEST_BAR);
        barWidth = constrain(barWidth, NARROWEST_BAR, WIDEST_BAR);
    }

    screen.beginDraw();
    screen.background(OFF);
    screen.stroke(ON);

    // Circle in top half — diameter changes with distance
    screen.fill(ON);
    screen.circle(6, 2, circleDiameter);

    // Rectangle at bottom — width changes with distance
    screen.noFill();
    screen.rect(0, 6, barWidth, 2);

    screen.endDraw();
}
```

**What changed:** Instead of one `map()` call, there are two — each converting the same sensor value into a different output range. The output range variables (`LARGEST_CIRCLE`, `WIDEST_BAR`) describe what each one controls. You can map one sensor to as many parameters as you want.

---

### Example: Sensor Controls Circle Position

Instead of changing the circle's **size**, the sensor changes its **x position** — the circle slides left and right across the screen.

```cpp
#include "TinyFilmFestival.h"
#include <EasyUltrasonic.h>

TinyScreen screen;
EasyUltrasonic ultrasonic;

int TRIG_PIN = A0;
int ECHO_PIN = A1;

int CLOSEST_CM = 2;
int FARTHEST_CM = 100;

// Output range is now screen position — named for what it means
int LEFT_EDGE = 0;
int RIGHT_EDGE = 11;

int READ_INTERVAL = 20;
unsigned long lastRead = 0;

float distanceToHand = 0;   // how far the hand is
int circleX = 0;             // where the circle sits on screen

void setup()
{
    Serial.begin(9600);
    screen.begin();
    ultrasonic.attach(TRIG_PIN, ECHO_PIN, CLOSEST_CM, FARTHEST_CM);
}

void loop()
{
    if (millis() - lastRead >= READ_INTERVAL)
    {
        lastRead = millis();
        distanceToHand = ultrasonic.getDistanceCM();

        // Map to x position — closer hand = further right
        circleX = map(distanceToHand, CLOSEST_CM, FARTHEST_CM, RIGHT_EDGE, LEFT_EDGE);
        circleX = constrain(circleX, LEFT_EDGE, RIGHT_EDGE);
    }

    screen.beginDraw();
    screen.background(OFF);
    screen.stroke(ON);
    screen.fill(ON);
    screen.circle(circleX, 4, 3);   // x moves, diameter is fixed at 3
    screen.endDraw();
}
```

**What changed:** The `map()` output feeds `circleX` (the x parameter of `screen.circle()`) instead of a diameter. The output range is named `LEFT_EDGE` / `RIGHT_EDGE` because that's what those numbers represent on screen. Same read → translate → draw pattern, different parameter.

---

### Example: Sensor Controls Oscillation Speed

This combines sensor input with `oscillateInt()` from Class 05. The circle bounces up and down automatically, but the **speed of the bounce** is controlled by distance — closer hand = faster bounce.

```cpp
#include "TinyFilmFestival.h"
#include <EasyUltrasonic.h>

TinyScreen screen;
EasyUltrasonic ultrasonic;

int TRIG_PIN = A0;
int ECHO_PIN = A1;

int CLOSEST_CM = 2;
int FARTHEST_CM = 100;

// Output range is oscillation period in milliseconds
// Named for what they mean: how fast is the fastest/slowest bounce
int FASTEST_BOUNCE = 200;     // ms — when hand is close
int SLOWEST_BOUNCE = 3000;    // ms — when hand is far

int READ_INTERVAL = 20;
unsigned long lastRead = 0;

float distanceToHand = 0;     // how far the hand is
int bounceSpeed = 1000;       // ms — how long one full bounce cycle takes

void setup()
{
    Serial.begin(9600);
    screen.begin();
    ultrasonic.attach(TRIG_PIN, ECHO_PIN, CLOSEST_CM, FARTHEST_CM);
}

void loop()
{
    if (millis() - lastRead >= READ_INTERVAL)
    {
        lastRead = millis();
        distanceToHand = ultrasonic.getDistanceCM();

        // Map hand distance to bounce speed
        // Closer hand = smaller period = faster bounce
        bounceSpeed = map(distanceToHand, CLOSEST_CM, FARTHEST_CM, FASTEST_BOUNCE, SLOWEST_BOUNCE);
        bounceSpeed = constrain(bounceSpeed, FASTEST_BOUNCE, SLOWEST_BOUNCE);
    }

    // oscillateInt handles the animation timing automatically
    // bounceSpeed controls how many ms one full cycle takes
    int yPos = oscillateInt(0, 7, bounceSpeed);

    screen.beginDraw();
    screen.background(OFF);
    screen.stroke(ON);
    screen.fill(ON);
    screen.circle(6, yPos, 3);   // y bounces, speed set by hand distance
    screen.endDraw();
}
```

**What changed:** The sensor doesn't control a shape property directly — it controls `bounceSpeed`, which is passed to `oscillateInt()` as the period. The range variables (`FASTEST_BOUNCE`, `SLOWEST_BOUNCE`) describe what those numbers mean in the interaction. The library handles the actual animation; the sensor just controls *how fast* it runs.

---

### Example: Multiple Oscillations at Different Speeds

Two circles bounce independently. The sensor controls the **speed ratio** between them — as one speeds up, the other slows down.

```cpp
#include "TinyFilmFestival.h"
#include <EasyUltrasonic.h>

TinyScreen screen;
EasyUltrasonic ultrasonic;

int TRIG_PIN = A0;
int ECHO_PIN = A1;

int CLOSEST_CM = 2;
int FARTHEST_CM = 100;

int READ_INTERVAL = 20;
unsigned long lastRead = 0;

float distanceToHand = 0;        // the shared input
int leftBounceSpeed = 1000;      // ms — period for left circle
int rightBounceSpeed = 1000;     // ms — period for right circle

void setup()
{
    Serial.begin(9600);
    screen.begin();
    ultrasonic.attach(TRIG_PIN, ECHO_PIN, CLOSEST_CM, FARTHEST_CM);
}

void loop()
{
    if (millis() - lastRead >= READ_INTERVAL)
    {
        lastRead = millis();
        distanceToHand = ultrasonic.getDistanceCM();

        // Left circle: closer hand = faster bounce (200–2000 ms)
        leftBounceSpeed = map(distanceToHand, CLOSEST_CM, FARTHEST_CM, 200, 2000);
        leftBounceSpeed = constrain(leftBounceSpeed, 200, 2000);

        // Right circle: closer hand = SLOWER bounce (opposite)
        rightBounceSpeed = map(distanceToHand, CLOSEST_CM, FARTHEST_CM, 2000, 200);
        rightBounceSpeed = constrain(rightBounceSpeed, 200, 2000);
    }

    // Two independent oscillations — each with its own speed
    int leftY = oscillateInt(0, 7, leftBounceSpeed);
    int rightY = oscillateInt(0, 7, rightBounceSpeed);

    screen.beginDraw();
    screen.background(OFF);
    screen.stroke(ON);
    screen.fill(ON);
    screen.circle(3, leftY, 3);     // left circle
    screen.circle(9, rightY, 3);    // right circle
    screen.endDraw();
}
```

**What changed:** Two `map()` calls from the same sensor, but with the output ranges **reversed**. `leftBounceSpeed` and `rightBounceSpeed` tell you exactly which circle each one controls. When the left speeds up, the right slows down. Each gets its own `oscillateInt()` call with its own period.

---

### Example: Distance Controls Fill Pattern

The sensor doesn't just control one circle — it fills a **row of dots** across the screen. Closer hand = more dots lit, like a progress bar.

```cpp
#include "TinyFilmFestival.h"
#include <EasyUltrasonic.h>

TinyScreen screen;
EasyUltrasonic ultrasonic;

int TRIG_PIN = A0;
int ECHO_PIN = A1;

int CLOSEST_CM = 2;
int FARTHEST_CM = 100;

int READ_INTERVAL = 20;
unsigned long lastRead = 0;

float distanceToHand = 0;   // how far the hand is
int dotsToLight = 0;         // how many dots to draw (0–12)

void setup()
{
    Serial.begin(9600);
    screen.begin();
    ultrasonic.attach(TRIG_PIN, ECHO_PIN, CLOSEST_CM, FARTHEST_CM);
}

void loop()
{
    if (millis() - lastRead >= READ_INTERVAL)
    {
        lastRead = millis();
        distanceToHand = ultrasonic.getDistanceCM();

        // Map to number of dots — closer hand = more dots
        dotsToLight = map(distanceToHand, CLOSEST_CM, FARTHEST_CM, 12, 0);
        dotsToLight = constrain(dotsToLight, 0, 12);
    }

    screen.beginDraw();
    screen.background(OFF);
    screen.stroke(ON);

    // Draw a row of dots — only up to dotsToLight
    for (int i = 0; i < dotsToLight; i++)
    {
        screen.point(i, 4);   // dot at column i, row 4
    }

    screen.endDraw();
}
```

**What changed:** Instead of controlling a single shape parameter, `dotsToLight` controls a `for` loop — it decides **how many** dots to draw. The variable name tells you exactly what it does. This shows that `map()` can drive any part of your code, not just the arguments inside a drawing function.

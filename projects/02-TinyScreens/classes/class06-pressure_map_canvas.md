# Pressure Sensor → Screen: Step by Step

[← Back to Class 06](class06-Feb13.md)

This walkthrough takes you from a bare sensor reading to a working on-screen interaction in **four steps**. Each step adds one idea and builds directly on the previous code. Don't skip ahead — the point is to understand what each piece does before the next one appears.

If you've already done the [Distance Sensor → Screen](class06-distance_map_canvas.md) walkthrough, this follows the exact same structure — only the sensor and variable names change.

---

## Step 1 — Read the Sensor

**Goal:** Confirm the pressure sensor is wired correctly and see the raw values it produces.

Build your custom FSR and wire it with the voltage divider circuit as shown in the [Class 06 wiring diagram](class06-Feb13.md#pressure-sensor--how-does-it-work). Then upload this sketch and open **Tools → Serial Monitor** at 9600 baud.

```cpp
// ============================================================
// Step 1 — Read the pressure sensor and print to Serial Monitor
// ============================================================
// What this does:
//   Reads the custom FSR (force-sensing resistor) every 20 ms
//   and prints the raw analog value to the Serial Monitor.
//
// Why this matters:
//   You need to SEE the real numbers your sensor produces
//   before you can do anything useful with them.
//   analogRead() returns 0–1023, but YOUR sensor's usable
//   range will be narrower depending on how it's built.
// ============================================================

// --- Pin configuration ---
// Must match the analog pin your FSR voltage divider is connected to
int FSR_PIN = A0;

// --- Timing ---
// We read the sensor on a timer instead of using delay()
// so the rest of the code can keep running smoothly.
int READ_INTERVAL = 20;       // ms between sensor reads
unsigned long lastRead = 0;   // tracks when we last read

// --- Sensor value ---
int pressureReading = 0;      // holds the most recent raw reading (0–1023)

void setup()
{
    Serial.begin(9600);
}

void loop()
{
    // Only read the sensor when enough time has passed
    if (millis() - lastRead >= READ_INTERVAL)
    {
        lastRead = millis();

        // Read the sensor — result is a raw analog value (0–1023)
        pressureReading = analogRead(FSR_PIN);

        // Print so we can see what the sensor is actually returning
        Serial.print("Pressure: ");
        Serial.println(pressureReading);
    }
}
```

### What to do with this

1. Upload the code and open the **Serial Monitor** (Tools → Serial Monitor, 9600 baud)
2. Press and release the sensor with varying amounts of force
3. **Write down two numbers:**
   - The **lightest press** value you see reliably — this is your **sensor minimum** (it won't be 0 — there's usually a baseline)
   - The **hardest press** value before it stops increasing — this is your **sensor maximum** (it usually won't reach 1023)

You'll use these numbers in the next step. Every custom FSR is different depending on materials, size, and construction, so **measure yours — don't guess**.

---

## Step 2 — Map the Value

**Goal:** Convert the raw sensor range into a useful output range and verify it with Serial Monitor.

In this step we add `map()` to translate the sensor's raw analog values into a new range: **1 to 12**. These will eventually become the diameter of a circle, but for now we just print both values so we can confirm the mapping works.

We also start **naming variables by what they represent** in the interaction — not generic labels like `val` or `mappedValue`, but names that describe the data: `gripStrength`, `LIGHTEST_PRESS`, `LARGEST_DIAMETER`. See [Input Language](class06-CodePatterns.md#input-language) for why this matters.

```cpp
// ============================================================
// Step 2 — Map the pressure to an output range
// ============================================================
// What's new:
//   - Variable names describe their purpose:
//     gripStrength, LIGHTEST_PRESS, LARGEST_DIAMETER, etc.
//   - LIGHTEST_PRESS / HARDEST_PRESS — your real measured range
//   - SMALLEST_DIAMETER / LARGEST_DIAMETER — the output range
//   - mappedSize — the result of the mapping
//   - map() converts one range to another
//   - constrain() clamps the result so it stays in bounds
//
// What to watch:
//   The Serial Monitor now prints BOTH values side by side
//   so you can see the mapping in action.
// ============================================================

// --- Pin configuration ---
int FSR_PIN = A0;

// --- Sensor range (from YOUR Serial Monitor observations) ---
// Replace these with the real min/max you wrote down in Step 1
// Name them for what they mean, not just "min" and "max"
int LIGHTEST_PRESS = 50;     // analog value at lightest usable press
int HARDEST_PRESS = 800;     // analog value at hardest press

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
int gripStrength = 0;         // how hard the sensor is being pressed (raw analog)
int mappedSize = 0;            // what diameter the map() produces

void setup()
{
    Serial.begin(9600);
}

void loop()
{
    if (millis() - lastRead >= READ_INTERVAL)
    {
        lastRead = millis();

        // READ — get the grip strength
        gripStrength = analogRead(FSR_PIN);

        // TRANSLATE — convert pressure → circle diameter
        // Harder press = bigger circle
        // map(value, fromLow, fromHigh, toLow, toHigh)
        mappedSize = map(gripStrength, LIGHTEST_PRESS, HARDEST_PRESS, SMALLEST_DIAMETER, LARGEST_DIAMETER);

        // constrain() keeps the result within bounds
        // in case the sensor returns a value outside our expected range
        mappedSize = constrain(mappedSize, SMALLEST_DIAMETER, LARGEST_DIAMETER);

        // Print both values so we can verify the mapping
        Serial.print("Grip strength: ");
        Serial.print(gripStrength);
        Serial.print(" → Circle diameter: ");
        Serial.println(mappedSize);
    }
}
```

### What to notice

- **`map()` direction is straightforward here.** `LIGHTEST_PRESS → SMALLEST_DIAMETER` and `HARDEST_PRESS → LARGEST_DIAMETER`. This means **harder press = bigger circle**. If you want the opposite, swap the diameter values in the `map()` call.
- **`constrain()`** is a safety net. If the sensor returns a value outside your expected range, `constrain()` clips it so `mappedSize` never goes below 1 or above 12.
- **Tuning:** If the numbers in the Serial Monitor don't spread across the full 1–12 range when you press, adjust `LIGHTEST_PRESS` and `HARDEST_PRESS` to better match what you actually see. These values vary a lot between custom FSRs.
- **Variable names tell the story.** Reading `gripStrength` and `LARGEST_DIAMETER` in the code, you immediately know what the data represents. Compare that to `pressure` and `OUTPUT_MAX` — those describe the *format* of the data but not its *meaning*.

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

TinyScreen screen;

// --- Pin configuration ---
int FSR_PIN = A0;

// --- Sensor range ---
int LIGHTEST_PRESS = 50;
int HARDEST_PRESS = 800;

// --- Output range ---
int SMALLEST_DIAMETER = 1;
int LARGEST_DIAMETER = 12;

// --- Timing ---
int READ_INTERVAL = 20;
unsigned long lastRead = 0;

// --- Values ---
int gripStrength = 0;         // how hard the sensor is being pressed
int mappedSize = 0;            // what diameter the map() produces

// --- Screen parameter ---
// This is the value that will eventually be driven by the sensor.
// For now it's fixed so we can confirm the circle draws correctly.
int circleDiameter = 6;       // fixed — we'll connect it in Step 4

void setup()
{
    Serial.begin(9600);
    screen.begin();
}

void loop()
{
    // --- Sensor reading (still running, still printing) ---
    if (millis() - lastRead >= READ_INTERVAL)
    {
        lastRead = millis();

        gripStrength = analogRead(FSR_PIN);
        mappedSize = map(gripStrength, LIGHTEST_PRESS, HARDEST_PRESS, SMALLEST_DIAMETER, LARGEST_DIAMETER);
        mappedSize = constrain(mappedSize, SMALLEST_DIAMETER, LARGEST_DIAMETER);

        Serial.print("Grip strength: ");
        Serial.print(gripStrength);
        Serial.print(" → Mapped diameter: ");
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
- The sensor is still reading and printing, but **nothing on screen changes yet**. The circle stays the same size no matter how hard you press.
- `screen.circle(6, 4, circleDiameter)` draws at roughly the center of the 12×8 LED matrix. The `6` is the x position (horizontal center) and `4` is the y position (vertical center).

---

## Step 4 — Connect Sensor to Screen

**Goal:** Replace the fixed diameter with the mapped sensor value so the circle responds to your grip.

This is the payoff. One line changes: `circleDiameter` is now computed from `map()` instead of being fixed at 6. The circle on screen grows and shrinks as you press harder and softer.

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

TinyScreen screen;

// --- Pin configuration ---
int FSR_PIN = A0;

// --- Sensor range (from YOUR Serial Monitor observations) ---
int LIGHTEST_PRESS = 50;     // analog value — update with your real minimum
int HARDEST_PRESS = 800;     // analog value — update with your real maximum

// --- Output range ---
int SMALLEST_DIAMETER = 1;      // smallest circle
int LARGEST_DIAMETER = 12;      // largest circle

// --- Timing ---
int READ_INTERVAL = 20;
unsigned long lastRead = 0;

// --- Values ---
int gripStrength = 0;            // how hard the sensor is being pressed
int circleDiameter = 0;          // now driven by the sensor

void setup()
{
    Serial.begin(9600);
    screen.begin();
}

void loop()
{
    // --- READ and TRANSLATE ---
    if (millis() - lastRead >= READ_INTERVAL)
    {
        lastRead = millis();

        // Read how hard the sensor is being pressed
        gripStrength = analogRead(FSR_PIN);

        // ★ THE CONNECTION — sensor drives the circle diameter directly
        // Harder press = bigger circle
        circleDiameter = map(gripStrength, LIGHTEST_PRESS, HARDEST_PRESS, SMALLEST_DIAMETER, LARGEST_DIAMETER);
        circleDiameter = constrain(circleDiameter, SMALLEST_DIAMETER, LARGEST_DIAMETER);

        // Still printing so you can debug
        Serial.print("Grip strength: ");
        Serial.print(gripStrength);
        Serial.print(" → Circle diameter: ");
        Serial.println(circleDiameter);
    }

    // --- DRAW ---
    screen.beginDraw();
    screen.background(OFF);
    screen.stroke(ON);
    screen.fill(ON);
    screen.circle(6, 4, circleDiameter);   // diameter changes with pressure!
    screen.endDraw();
}
```

### Tuning the Interaction

The code works, but the *feel* of the interaction depends on the numbers. Here's what to adjust:

| What to change | Where | Effect |
|---|---|---|
| **Sensor range** | `LIGHTEST_PRESS` / `HARDEST_PRESS` | Narrower range = more sensitive to small changes. If your sensor reads 100–500 in practice, use those values. |
| **Output range** | `SMALLEST_DIAMETER` / `LARGEST_DIAMETER` | Change the smallest and largest circle sizes. Try `3` to `10` for a subtler effect. |
| **Map direction** | Swap `SMALLEST_DIAMETER` and `LARGEST_DIAMETER` in `map()` | Reverses the interaction — harder press = smaller instead of bigger. |
| **Circle position** | `screen.circle(6, 4, ...)` | Move the circle to a different spot on the matrix. `(0, 0)` = top-left, `(11, 7)` = bottom-right. |
| **Read speed** | `READ_INTERVAL` | Lower = more responsive but potentially jittery. Higher = smoother but slower to react. |

### What's Next?

You now have the core pattern: **read → translate → draw**. Every sensor example in [Class 06](class06-Feb13.md) follows this same structure. From here you can:

- Control **animation speed** instead of circle size — see [Pressure Controls Animation Speed](class06-Feb13.md#example-pressure-controls-animation-speed)
- **Switch between animations** at a threshold — see [Pressure Threshold](class06-Feb13.md#example-pressure-threshold--switch-animations)
- Control **position** instead of size — see [Pressure Mapped to Canvas](class06-Feb13.md#example-pressure-mapped-to-canvas)
- Try the same pattern with a **distance sensor** — see [Distance Sensor → Screen](class06-distance_map_canvas.md)

---

## Going Further

The examples below all start from the Step 4 code. Each one changes or adds a small amount to show a different way to use the same sensor data. Comments in the code highlight what's different.

---

### Example: Two Shapes from One Sensor

The sensor controls **two things at once** — a circle's diameter and a rectangle's height. Both use `gripStrength`, but each maps it to a different output range with a name that describes what it controls.

```cpp
#include "TinyFilmFestival.h"

TinyScreen screen;

int FSR_PIN = A0;

// --- Sensor range ---
int LIGHTEST_PRESS = 50;
int HARDEST_PRESS = 800;

// --- Output ranges — named for what each shape needs ---
int SMALLEST_CIRCLE = 1;
int LARGEST_CIRCLE = 6;
int SHORTEST_BAR = 0;
int TALLEST_BAR = 8;

int READ_INTERVAL = 20;
unsigned long lastRead = 0;

int gripStrength = 0;           // the input
int circleDiameter = 0;         // output: how big the circle is
int barHeight = 0;               // output: how tall the rectangle is

void setup()
{
    Serial.begin(9600);
    screen.begin();
}

void loop()
{
    if (millis() - lastRead >= READ_INTERVAL)
    {
        lastRead = millis();
        gripStrength = analogRead(FSR_PIN);

        // Same sensor, two different map() calls — each named for its purpose
        circleDiameter = map(gripStrength, LIGHTEST_PRESS, HARDEST_PRESS, SMALLEST_CIRCLE, LARGEST_CIRCLE);
        circleDiameter = constrain(circleDiameter, SMALLEST_CIRCLE, LARGEST_CIRCLE);

        barHeight = map(gripStrength, LIGHTEST_PRESS, HARDEST_PRESS, SHORTEST_BAR, TALLEST_BAR);
        barHeight = constrain(barHeight, SHORTEST_BAR, TALLEST_BAR);
    }

    screen.beginDraw();
    screen.background(OFF);
    screen.stroke(ON);

    // Circle on the right — diameter grows with pressure
    screen.fill(ON);
    screen.circle(9, 4, circleDiameter);

    // Bar on the left — grows upward from the bottom with pressure
    screen.noFill();
    screen.rect(0, 8 - barHeight, 3, barHeight);

    screen.endDraw();
}
```

**What changed:** Instead of one `map()` call, there are two — each converting the same sensor value into a different output range. The output range variables (`LARGEST_CIRCLE`, `TALLEST_BAR`) describe what each one controls. You can map one sensor to as many parameters as you want.

---

### Example: Pressure Controls Circle Position

Instead of changing the circle's **size**, the sensor changes its **y position** — the circle moves up and down on the screen as you press harder.

```cpp
#include "TinyFilmFestival.h"

TinyScreen screen;

int FSR_PIN = A0;

int LIGHTEST_PRESS = 50;
int HARDEST_PRESS = 800;

// Output range is now screen position — named for what it means
int BOTTOM_EDGE = 7;
int TOP_EDGE = 0;

int READ_INTERVAL = 20;
unsigned long lastRead = 0;

int gripStrength = 0;       // how hard the sensor is being pressed
int circleY = 0;              // where the circle sits on screen

void setup()
{
    Serial.begin(9600);
    screen.begin();
}

void loop()
{
    if (millis() - lastRead >= READ_INTERVAL)
    {
        lastRead = millis();
        gripStrength = analogRead(FSR_PIN);

        // Map to y position — harder press = higher on screen (lower y value)
        circleY = map(gripStrength, LIGHTEST_PRESS, HARDEST_PRESS, BOTTOM_EDGE, TOP_EDGE);
        circleY = constrain(circleY, TOP_EDGE, BOTTOM_EDGE);
    }

    screen.beginDraw();
    screen.background(OFF);
    screen.stroke(ON);
    screen.fill(ON);
    screen.circle(6, circleY, 3);   // y moves, diameter is fixed at 3
    screen.endDraw();
}
```

**What changed:** The `map()` output feeds `circleY` (the y parameter of `screen.circle()`) instead of a diameter. The output range is named `BOTTOM_EDGE` / `TOP_EDGE` because that's what those numbers represent on screen. Harder press pushes the circle upward.

---

### Example: Pressure Controls Oscillation Speed

This combines sensor input with `oscillateInt()` from Class 05. The circle bounces left and right automatically, but the **speed of the bounce** is controlled by pressure — harder press = faster bounce.

```cpp
#include "TinyFilmFestival.h"

TinyScreen screen;

int FSR_PIN = A0;

int LIGHTEST_PRESS = 50;
int HARDEST_PRESS = 800;

// Output range is oscillation period in milliseconds
// Named for what they mean: how fast is the fastest/slowest bounce
int SLOWEST_BOUNCE = 3000;   // ms — when barely pressing
int FASTEST_BOUNCE = 200;    // ms — when pressing hard

int READ_INTERVAL = 20;
unsigned long lastRead = 0;

int gripStrength = 0;         // how hard the sensor is being pressed
int bounceSpeed = 1000;       // ms — how long one full bounce cycle takes

void setup()
{
    Serial.begin(9600);
    screen.begin();
}

void loop()
{
    if (millis() - lastRead >= READ_INTERVAL)
    {
        lastRead = millis();
        gripStrength = analogRead(FSR_PIN);

        // Map grip strength to bounce speed
        // Harder press = smaller period = faster bounce
        bounceSpeed = map(gripStrength, LIGHTEST_PRESS, HARDEST_PRESS, SLOWEST_BOUNCE, FASTEST_BOUNCE);
        bounceSpeed = constrain(bounceSpeed, FASTEST_BOUNCE, SLOWEST_BOUNCE);
    }

    // oscillateInt handles the animation timing automatically
    // bounceSpeed controls how many ms one full cycle takes
    int xPos = oscillateInt(0, 11, bounceSpeed);

    screen.beginDraw();
    screen.background(OFF);
    screen.stroke(ON);
    screen.fill(ON);
    screen.circle(xPos, 4, 3);   // x bounces, speed set by grip strength
    screen.endDraw();
}
```

**What changed:** The sensor doesn't control a shape property directly — it controls `bounceSpeed`, which is passed to `oscillateInt()` as the period. The range variables (`FASTEST_BOUNCE`, `SLOWEST_BOUNCE`) describe what those numbers mean in the interaction. The library handles the actual animation; the sensor just controls *how fast* it runs.

---

### Example: Multiple Oscillations at Different Speeds

Two circles bounce independently. The sensor controls the **speed ratio** between them — as one speeds up, the other slows down.

```cpp
#include "TinyFilmFestival.h"

TinyScreen screen;

int FSR_PIN = A0;

int LIGHTEST_PRESS = 50;
int HARDEST_PRESS = 800;

int READ_INTERVAL = 20;
unsigned long lastRead = 0;

int gripStrength = 0;              // the shared input
int topBounceSpeed = 1000;         // ms — period for top circle
int bottomBounceSpeed = 1000;      // ms — period for bottom circle

void setup()
{
    Serial.begin(9600);
    screen.begin();
}

void loop()
{
    if (millis() - lastRead >= READ_INTERVAL)
    {
        lastRead = millis();
        gripStrength = analogRead(FSR_PIN);

        // Top circle: harder press = faster bounce (200–2000 ms)
        topBounceSpeed = map(gripStrength, LIGHTEST_PRESS, HARDEST_PRESS, 2000, 200);
        topBounceSpeed = constrain(topBounceSpeed, 200, 2000);

        // Bottom circle: harder press = SLOWER bounce (opposite)
        bottomBounceSpeed = map(gripStrength, LIGHTEST_PRESS, HARDEST_PRESS, 200, 2000);
        bottomBounceSpeed = constrain(bottomBounceSpeed, 200, 2000);
    }

    // Two independent oscillations — each with its own speed
    int topX = oscillateInt(0, 11, topBounceSpeed);
    int bottomX = oscillateInt(0, 11, bottomBounceSpeed);

    screen.beginDraw();
    screen.background(OFF);
    screen.stroke(ON);
    screen.fill(ON);
    screen.circle(topX, 2, 3);        // top circle
    screen.circle(bottomX, 6, 3);     // bottom circle
    screen.endDraw();
}
```

**What changed:** Two `map()` calls from the same sensor, but with the output ranges **reversed**. `topBounceSpeed` and `bottomBounceSpeed` tell you exactly which circle each one controls. When the top speeds up, the bottom slows down. Each gets its own `oscillateInt()` call with its own period.

---

### Example: Pressure Controls Fill Pattern

The sensor fills a **column of dots** from bottom to top. Harder press = more dots lit, like a meter.

```cpp
#include "TinyFilmFestival.h"

TinyScreen screen;

int FSR_PIN = A0;

int LIGHTEST_PRESS = 50;
int HARDEST_PRESS = 800;

int READ_INTERVAL = 20;
unsigned long lastRead = 0;

int gripStrength = 0;       // how hard the sensor is being pressed
int dotsToLight = 0;         // how many dots to draw (0–8)

void setup()
{
    Serial.begin(9600);
    screen.begin();
}

void loop()
{
    if (millis() - lastRead >= READ_INTERVAL)
    {
        lastRead = millis();
        gripStrength = analogRead(FSR_PIN);

        // Map to number of dots — harder press = more dots
        dotsToLight = map(gripStrength, LIGHTEST_PRESS, HARDEST_PRESS, 0, 8);
        dotsToLight = constrain(dotsToLight, 0, 8);
    }

    screen.beginDraw();
    screen.background(OFF);
    screen.stroke(ON);

    // Draw a column of dots from bottom up — only up to dotsToLight
    for (int i = 0; i < dotsToLight; i++)
    {
        screen.point(6, 7 - i);   // dot at column 6, counting up from bottom
    }

    screen.endDraw();
}
```

**What changed:** Instead of controlling a single shape parameter, `dotsToLight` controls a `for` loop — it decides **how many** dots to draw. The column fills from the bottom upward, which feels natural for a pressure meter. This shows that `map()` can drive any part of your code, not just the arguments inside a drawing function.

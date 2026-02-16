# Distance Thresholds → Shapes: Step by Step

[← Back to Class 06](class06-Feb13.md)

This walkthrough takes you from a generic sensor reading to a threshold-based interaction in **five steps**. Instead of continuously mapping a sensor value to a single parameter (like the [Distance → Screen](class06-distance_map_canvas.md) walkthrough does), here you **divide the sensor range into zones** and draw a different shape in each zone.

The scenario: the distance sensor is pointing down at a table, and you're **lifting the object up**. The sensor measures how high above the table the object is, and the screen changes what it shows at different heights.

---

## Step 1 — Read the Sensor

**Goal:** Confirm the sensor is wired correctly and see the raw values it produces.

Wire the HC-SR04 as shown in the [Class 06 wiring diagram](class06-Feb13.md#distance-sensor--how-does-it-work). Point the sensor **downward** toward the table surface. Then upload this sketch and open **Tools → Serial Monitor** at 9600 baud.

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
//   The sensor is pointing down — the distance it reads
//   is the height above the table.
// ============================================================

#include <EasyUltrasonic.h>

EasyUltrasonic ultrasonic;

// --- Pin configuration ---
int TRIG_PIN = A0;
int ECHO_PIN = A1;

// --- Sensor range ---
int MIN_DISTANCE = 2;    // cm — closest reliable reading
int MAX_DISTANCE = 100;  // cm — farthest we care about

// --- Timing ---
int READ_INTERVAL = 20;
unsigned long lastRead = 0;

// --- Sensor value ---
float heightAboveTable = 0;   // how high the object is lifted (cm)

void setup()
{
    Serial.begin(9600);
    ultrasonic.attach(TRIG_PIN, ECHO_PIN, MIN_DISTANCE, MAX_DISTANCE);
}

void loop()
{
    if (millis() - lastRead >= READ_INTERVAL)
    {
        lastRead = millis();

        heightAboveTable = ultrasonic.getDistanceCM();

        Serial.print("Height above table (cm): ");
        Serial.println(heightAboveTable);
    }
}
```

### What to do with this

1. Upload the code and open the **Serial Monitor** (Tools → Serial Monitor, 9600 baud).
2. Place the sensor face-down on a surface, or hold it pointing downward.
3. Slowly lift the object upward and watch the numbers change.
4. **Write down three things:**
   - The value when the object is sitting on the table (very close) — your **minimum**
   - The value when it's lifted as high as you'd reasonably hold it — your **maximum**
   - Roughly where the **middle** of that range falls

You'll use these numbers to define your threshold boundaries in Step 2.

---

## Step 2 — Define the Thresholds

**Goal:** Split the sensor range into three named zones and print which zone the object is in.

Instead of using `map()` to create a continuous output, we use `if / else if / else` to divide the range into **discrete zones**. Each zone gets a descriptive name that matches the interaction: `LOW`, `MID`, `HIGH`.

```cpp
// ============================================================
// Step 2 — Divide the height into three zones
// ============================================================
// What's new:
//   - Three threshold boundaries split the range into zones
//   - if / else if / else determines which zone the object is in
//   - currentZone tracks the zone as a number (1, 2, 3)
//   - Serial Monitor prints both the height and the zone name
//
// Why thresholds instead of map():
//   Sometimes you don't want a smooth gradient — you want
//   distinct states. "On the table," "lifted partway,"
//   and "held high" are three different states, not a spectrum.
// ============================================================

#include <EasyUltrasonic.h>

EasyUltrasonic ultrasonic;

int TRIG_PIN = A0;
int ECHO_PIN = A1;

// --- Sensor range ---
int MIN_DISTANCE = 2;
int MAX_DISTANCE = 60;

// --- Threshold boundaries (cm) ---
// These divide the sensor range into three zones.
// Adjust these based on YOUR Serial Monitor observations from Step 1.
int LOW_TO_MID = 15;      // below this = LOW zone (on/near the table)
int MID_TO_HIGH = 35;     // above this = HIGH zone (held up high)
                           // between LOW_TO_MID and MID_TO_HIGH = MID zone

// --- Timing ---
int READ_INTERVAL = 20;
unsigned long lastRead = 0;

// --- Values ---
float heightAboveTable = 0;
int currentZone = 0;       // 1 = LOW, 2 = MID, 3 = HIGH

void setup()
{
    Serial.begin(9600);
    ultrasonic.attach(TRIG_PIN, ECHO_PIN, MIN_DISTANCE, MAX_DISTANCE);
}

void loop()
{
    if (millis() - lastRead >= READ_INTERVAL)
    {
        lastRead = millis();
        heightAboveTable = ultrasonic.getDistanceCM();

        // Determine which zone the object is in
        if (heightAboveTable < LOW_TO_MID)
        {
            currentZone = 1;  // LOW — on or near the table
        }
        else if (heightAboveTable < MID_TO_HIGH)
        {
            currentZone = 2;  // MID — lifted partway
        }
        else
        {
            currentZone = 3;  // HIGH — held up high
        }

        // Print both values so we can verify the zones
        Serial.print("Height: ");
        Serial.print(heightAboveTable);
        Serial.print(" cm → Zone: ");
        Serial.println(currentZone);
    }
}
```

### What to notice

- **Two boundaries create three zones.** `LOW_TO_MID` and `MID_TO_HIGH` are all you need. Everything below the first is LOW, everything above the second is HIGH, everything between is MID.
- **The zone names describe the interaction.** Reading `LOW_TO_MID = 15` tells you that below 15 cm the object is considered near the table. Compare that to `THRESHOLD_1 = 15` — same number, but no meaning.
- **`currentZone` is a simple number.** Using 1, 2, 3 makes it easy to use in `if` statements later. In Step 3 we'll use this number to choose which shape to draw.
- **Tuning:** If the Serial Monitor shows the object jumping between zones erratically, widen the gap between thresholds. If a zone never triggers, adjust the boundary values to match your actual sensor range.

---

## Step 3 — Draw Three Static Shapes

**Goal:** Get the TinyFilmFestival library drawing three different shapes on the LED matrix, one at a time, controlled by a hard-coded zone number.

Before connecting the sensor, we set up the drawing code with a **fixed `currentZone`** value. This follows the [From Static to Dynamic](class06-CodePatterns.md#from-static-to-dynamic) pattern — get the output working first, then connect it to input.

```cpp
// ============================================================
// Step 3 — Draw a different shape for each zone (static)
// ============================================================
// What's new:
//   - TinyFilmFestival library added
//   - Three shapes: dot (LOW), circle (MID), cross (HIGH)
//   - if / else if / else picks the right shape based on zone
//   - currentZone is fixed at 2 — change it to 1 or 3 to
//     test each shape before connecting the sensor
//
// The sensor is still here and printing, but the shapes
// are NOT connected to it yet.
// ============================================================

#include "TinyFilmFestival.h"
#include <EasyUltrasonic.h>

TinyScreen screen;
EasyUltrasonic ultrasonic;

int TRIG_PIN = A0;
int ECHO_PIN = A1;

int MIN_DISTANCE = 2;
int MAX_DISTANCE = 60;

int LOW_TO_MID = 15;
int MID_TO_HIGH = 35;

int READ_INTERVAL = 20;
unsigned long lastRead = 0;

float heightAboveTable = 0;

// --- Zone ---
// Fixed for now — change this to 1, 2, or 3 to test each shape
int currentZone = 2;

void setup()
{
    Serial.begin(9600);
    screen.begin();
    ultrasonic.attach(TRIG_PIN, ECHO_PIN, MIN_DISTANCE, MAX_DISTANCE);
}

void loop()
{
    // --- Sensor reading (still printing, not connected to shapes) ---
    if (millis() - lastRead >= READ_INTERVAL)
    {
        lastRead = millis();
        heightAboveTable = ultrasonic.getDistanceCM();

        Serial.print("Height: ");
        Serial.print(heightAboveTable);
        Serial.print(" cm (zone fixed at ");
        Serial.print(currentZone);
        Serial.println(")");
    }

    // --- Draw the shape for the current zone ---
    screen.beginDraw();
    screen.background(OFF);
    screen.stroke(ON);

    if (currentZone == 1)
    {
        // LOW — small dot in the center (resting on table)
        screen.point(6, 4);
    }
    else if (currentZone == 2)
    {
        // MID — filled circle (lifted partway)
        screen.fill(ON);
        screen.circle(6, 4, 5);
    }
    else
    {
        // HIGH — cross pattern (held up high)
        screen.line(0, 4, 11, 4);  // horizontal line
        screen.line(6, 0, 6, 7);   // vertical line
    }

    screen.endDraw();
}
```

### What to notice

- The `if / else if / else` block inside `loop()` picks which shape to draw based on `currentZone`. This is the same pattern from Step 2, but now it controls drawing instead of printing.
- **Change `currentZone` to 1, 2, or 3**, upload, and confirm each shape looks right before connecting the sensor. This is the static-to-dynamic pattern in action.
- The three shapes have increasing visual weight: a single dot → a filled circle → a full cross. This creates a clear visual progression as you lift higher.
- The sensor is still reading and printing, but nothing on screen changes yet.
- Notice the drawing code is mixed in with the rest of `loop()`. In Step 4 we'll clean this up by moving the shapes into their own function.

---

## Step 4 — Organize with a Function

**Goal:** Move the shape-drawing code into its own function so `loop()` is easier to read.

The code does exactly the same thing as Step 3 — nothing new happens on screen. The only change is **where** the drawing code lives. Instead of sitting inside `loop()`, the three `if / else if / else` blocks move into a function called `drawShape()`. For more on why and how functions work, see the [Writing Functions](class06-CodePatterns.md#writing-functions) section in the Code Patterns reference.

```cpp
// ============================================================
// Step 4 — Same shapes, organized into a function
// ============================================================
// What changed:
//   The if / else if / else drawing code from Step 3 is now
//   inside a drawShape() function. loop() calls drawShape()
//   instead of containing the drawing logic directly.
//
//   Behavior is identical — this is a structural improvement.
// ============================================================

#include "TinyFilmFestival.h"
#include <EasyUltrasonic.h>

TinyScreen screen;
EasyUltrasonic ultrasonic;

int TRIG_PIN = A0;
int ECHO_PIN = A1;

int MIN_DISTANCE = 2;
int MAX_DISTANCE = 60;

int LOW_TO_MID = 15;
int MID_TO_HIGH = 35;

int READ_INTERVAL = 20;
unsigned long lastRead = 0;

float heightAboveTable = 0;

// --- Zone ---
// Still fixed for testing — we'll connect the sensor in Step 5
int currentZone = 2;

void setup()
{
    Serial.begin(9600);
    screen.begin();
    ultrasonic.attach(TRIG_PIN, ECHO_PIN, MIN_DISTANCE, MAX_DISTANCE);
}

// --- Draw the shape for a given zone ---
// This is the same if / else if / else from Step 3,
// pulled out into its own function.
void drawShape(int zone)
{
    if (zone == 1)
    {
        // LOW — small dot in the center (resting on table)
        screen.point(6, 4);
    }
    else if (zone == 2)
    {
        // MID — filled circle (lifted partway)
        screen.fill(ON);
        screen.circle(6, 4, 5);
    }
    else
    {
        // HIGH — cross pattern (held up high)
        screen.line(0, 4, 11, 4);  // horizontal line
        screen.line(6, 0, 6, 7);   // vertical line
    }
}

void loop()
{
    // --- Sensor reading (still printing, not connected to shapes) ---
    if (millis() - lastRead >= READ_INTERVAL)
    {
        lastRead = millis();
        heightAboveTable = ultrasonic.getDistanceCM();

        Serial.print("Height: ");
        Serial.print(heightAboveTable);
        Serial.print(" cm (zone fixed at ");
        Serial.print(currentZone);
        Serial.println(")");
    }

    // --- Draw ---
    screen.beginDraw();
    screen.background(OFF);
    screen.stroke(ON);
    drawShape(currentZone);
    screen.endDraw();
}
```

### What to notice

- **`drawShape()` is a function** that takes a zone number and draws the matching shape. It uses `void` because it performs an action and doesn't need to return a value. See [Types of Functions](class06-CodePatterns.md#types-of-functions) for more on this.
- **`loop()` now reads like a plan:** read the sensor, draw the shape. The details of *which* shape and *how* to draw it are tucked away inside `drawShape()`.
- **The behavior is identical to Step 3.** Functions don't add new features — they organize existing code so it's easier to read, modify, and reuse.
- As your sketch grows (more zones, more complex shapes), having drawing code in its own function keeps `loop()` manageable.

---

## Step 5 — Connect Sensor to Shapes

**Goal:** Replace the fixed zone number with the threshold logic from Step 2 so the shape on screen changes as you lift the object.

One section changes: `currentZone` is now computed from the sensor value using `if / else if / else`, instead of being fixed at 2.

```cpp
// ============================================================
// Step 5 — Sensor height controls which shape is drawn
// ============================================================
// What changed:
//   currentZone is now computed from heightAboveTable using
//   the threshold boundaries. The drawShape() function from
//   Step 4 handles the rest — lift to see the shapes change.
// ============================================================

#include "TinyFilmFestival.h"
#include <EasyUltrasonic.h>

TinyScreen screen;
EasyUltrasonic ultrasonic;

// --- Pin configuration ---
int TRIG_PIN = A0;
int ECHO_PIN = A1;

// --- Sensor range ---
int MIN_DISTANCE = 2;
int MAX_DISTANCE = 60;

// --- Threshold boundaries (from YOUR observations) ---
int LOW_TO_MID = 15;      // below = on the table
int MID_TO_HIGH = 35;     // above = held up high

// --- Timing ---
int READ_INTERVAL = 20;
unsigned long lastRead = 0;

// --- Values ---
float heightAboveTable = 0;
int currentZone = 1;       // now driven by the sensor

void setup()
{
    Serial.begin(9600);
    screen.begin();
    ultrasonic.attach(TRIG_PIN, ECHO_PIN, MIN_DISTANCE, MAX_DISTANCE);
}

// --- Draw the shape for a given zone ---
void drawShape(int zone)
{
    if (zone == 1)
    {
        // LOW — small dot (resting on table)
        screen.point(6, 4);
    }
    else if (zone == 2)
    {
        // MID — filled circle (lifted partway)
        screen.fill(ON);
        screen.circle(6, 4, 5);
    }
    else
    {
        // HIGH — cross pattern (held up high)
        screen.line(0, 4, 11, 4);
        screen.line(6, 0, 6, 7);
    }
}

void loop()
{
    if (millis() - lastRead >= READ_INTERVAL)
    {
        lastRead = millis();
        heightAboveTable = ultrasonic.getDistanceCM();

        // ★ THE CONNECTION — height determines the zone
        if (heightAboveTable < LOW_TO_MID)
        {
            currentZone = 1;  // LOW
        }
        else if (heightAboveTable < MID_TO_HIGH)
        {
            currentZone = 2;  // MID
        }
        else
        {
            currentZone = 3;  // HIGH
        }

        Serial.print("Height: ");
        Serial.print(heightAboveTable);
        Serial.print(" cm → Zone ");
        Serial.println(currentZone);
    }

    // --- DRAW ---
    screen.beginDraw();
    screen.background(OFF);
    screen.stroke(ON);
    drawShape(currentZone);
    screen.endDraw();
}
```

### Tuning the Interaction

| What to change | Where | Effect |
|---|---|---|
| **Zone boundaries** | `LOW_TO_MID` / `MID_TO_HIGH` | Shift where the transitions happen. Closer values = smaller MID zone. |
| **Shapes** | Inside `drawShape()` | Swap in any shapes you want — `screen.rect()`, `screen.circle()`, `screen.line()`, `screen.point()`. |
| **Number of zones** | Add more `else if` blocks | You can have 4, 5, or more zones — just add more threshold boundaries and shapes. |
| **Dead zone** | Add a gap between thresholds | If the display flickers at a boundary, leave a small gap (e.g., LOW < 14, MID > 16) — values in the gap keep the previous zone. |

### What's Next?

You now have the core pattern: **read → classify → draw**. This is different from the `map()` pattern — instead of smooth, continuous output, you get distinct **states**. From here you can:

- Add **continuous mapping within each zone** — see Going Further below
- Control **animations per zone** instead of shapes — see [Distance → Animation](class06-distance_map_animation.md)
- Try the same threshold pattern with a **pressure sensor** — see [Pressure → Screen](class06-pressure_map_canvas.md)

---

## Going Further

The examples below all start from the Step 5 code. Each one extends the threshold idea in a different direction. Comments highlight what's different.

---

### Example: Size Changes Within Each Zone

Thresholds pick the shape, but `map()` controls the **size within each zone**. The shape changes at the boundary, and within each zone the shape grows as you lift higher.

```cpp
#include "TinyFilmFestival.h"
#include <EasyUltrasonic.h>

TinyScreen screen;
EasyUltrasonic ultrasonic;

int TRIG_PIN = A0;
int ECHO_PIN = A1;

int MIN_DISTANCE = 2;
int MAX_DISTANCE = 60;

// --- Threshold boundaries ---
int LOW_TO_MID = 15;
int MID_TO_HIGH = 35;

int READ_INTERVAL = 20;
unsigned long lastRead = 0;

float heightAboveTable = 0;
int currentZone = 1;
int shapeSize = 1;         // size parameter within each zone

void setup()
{
    Serial.begin(9600);
    screen.begin();
    ultrasonic.attach(TRIG_PIN, ECHO_PIN, MIN_DISTANCE, MAX_DISTANCE);
}

void drawShape(int zone, int size)
{
    if (zone == 1)
    {
        // LOW — dot cluster, size controls how many dots
        int dots = constrain(size, 1, 4);
        screen.point(6, 4);
        if (dots >= 2) screen.point(5, 4);
        if (dots >= 3) screen.point(7, 4);
        if (dots >= 4) { screen.point(6, 3); screen.point(6, 5); }
    }
    else if (zone == 2)
    {
        // MID — circle, size controls diameter
        screen.fill(ON);
        screen.circle(6, 4, size);
    }
    else
    {
        // HIGH — cross, size controls arm length
        int halfLength = constrain(size / 2, 1, 6);
        screen.line(6 - halfLength, 4, 6 + halfLength, 4);
        screen.line(6, 4 - halfLength, 6, 4 + halfLength);
    }
}

void loop()
{
    if (millis() - lastRead >= READ_INTERVAL)
    {
        lastRead = millis();
        heightAboveTable = ultrasonic.getDistanceCM();

        if (heightAboveTable < LOW_TO_MID)
        {
            currentZone = 1;
            // Map within the LOW zone: 2–15 cm → size 1–4
            shapeSize = map(heightAboveTable, MIN_DISTANCE, LOW_TO_MID, 1, 4);
            shapeSize = constrain(shapeSize, 1, 4);
        }
        else if (heightAboveTable < MID_TO_HIGH)
        {
            currentZone = 2;
            // Map within the MID zone: 15–35 cm → diameter 2–8
            shapeSize = map(heightAboveTable, LOW_TO_MID, MID_TO_HIGH, 2, 8);
            shapeSize = constrain(shapeSize, 2, 8);
        }
        else
        {
            currentZone = 3;
            // Map within the HIGH zone: 35–60 cm → arm length 4–12
            shapeSize = map(heightAboveTable, MID_TO_HIGH, MAX_DISTANCE, 4, 12);
            shapeSize = constrain(shapeSize, 4, 12);
        }
    }

    screen.beginDraw();
    screen.background(OFF);
    screen.stroke(ON);
    drawShape(currentZone, shapeSize);
    screen.endDraw();
}
```

**What changed:** `drawShape()` now takes a `size` parameter in addition to `zone`. Each `if` block does its own `map()` call using only the range within that zone. This combines thresholds (discrete shape changes) with continuous mapping (smooth size changes within each shape).

---

### Example: Adding a Bounce to Each Zone

Each zone's shape **oscillates** at a different speed. The LOW zone barely moves, MID bounces gently, and HIGH bounces fast — the higher you lift, the more energetic the motion.

```cpp
#include "TinyFilmFestival.h"
#include <EasyUltrasonic.h>

TinyScreen screen;
EasyUltrasonic ultrasonic;

int TRIG_PIN = A0;
int ECHO_PIN = A1;

int MIN_DISTANCE = 2;
int MAX_DISTANCE = 60;

int LOW_TO_MID = 15;
int MID_TO_HIGH = 35;

// --- Bounce speed per zone (ms per cycle) ---
int LOW_BOUNCE = 4000;     // very slow drift
int MID_BOUNCE = 1500;     // gentle bounce
int HIGH_BOUNCE = 400;     // fast bounce

int READ_INTERVAL = 20;
unsigned long lastRead = 0;

float heightAboveTable = 0;
int currentZone = 1;
int bounceSpeed = 4000;

void setup()
{
    Serial.begin(9600);
    screen.begin();
    ultrasonic.attach(TRIG_PIN, ECHO_PIN, MIN_DISTANCE, MAX_DISTANCE);
}

void loop()
{
    if (millis() - lastRead >= READ_INTERVAL)
    {
        lastRead = millis();
        heightAboveTable = ultrasonic.getDistanceCM();

        if (heightAboveTable < LOW_TO_MID)
        {
            currentZone = 1;
            bounceSpeed = LOW_BOUNCE;
        }
        else if (heightAboveTable < MID_TO_HIGH)
        {
            currentZone = 2;
            bounceSpeed = MID_BOUNCE;
        }
        else
        {
            currentZone = 3;
            bounceSpeed = HIGH_BOUNCE;
        }
    }

    // Y position oscillates — speed depends on which zone
    int yPos = oscillateInt(1, 6, bounceSpeed);

    screen.beginDraw();
    screen.background(OFF);
    screen.stroke(ON);

    if (currentZone == 1)
    {
        screen.point(6, yPos);
    }
    else if (currentZone == 2)
    {
        screen.fill(ON);
        screen.circle(6, yPos, 5);
    }
    else
    {
        screen.line(0, yPos, 11, yPos);
        screen.line(6, 0, 6, 7);
    }

    screen.endDraw();
}
```

**What changed:** Each zone now has a fixed `bounceSpeed` that gets passed to `oscillateInt()`. The shape still changes at the thresholds, but within each zone the shape bounces vertically. The bounce speed jumps discretely at each boundary — LOW drifts slowly, MID bounces, HIGH jitters.

---

### Example: Four Zones with Dead Zones

Add a **fourth zone** and introduce **dead zones** — small gaps between thresholds where the display keeps its current state. This prevents flickering when the sensor value hovers near a boundary.

```cpp
#include "TinyFilmFestival.h"
#include <EasyUltrasonic.h>

TinyScreen screen;
EasyUltrasonic ultrasonic;

int TRIG_PIN = A0;
int ECHO_PIN = A1;

int MIN_DISTANCE = 2;
int MAX_DISTANCE = 60;

// --- Threshold boundaries with dead zones ---
// Zone 1 (GROUND):  below 10
// Dead zone:         10–12  (keeps current zone)
// Zone 2 (LOW):      12–25
// Dead zone:         25–28  (keeps current zone)
// Zone 3 (MID):      28–42
// Dead zone:         42–45  (keeps current zone)
// Zone 4 (HIGH):     above 45

int GROUND_LOW_ENTER = 12;    // must rise above 12 to enter LOW
int GROUND_LOW_EXIT = 10;     // must drop below 10 to return to GROUND
int LOW_MID_ENTER = 28;
int LOW_MID_EXIT = 25;
int MID_HIGH_ENTER = 45;
int MID_HIGH_EXIT = 42;

int READ_INTERVAL = 20;
unsigned long lastRead = 0;

float heightAboveTable = 0;
int currentZone = 1;

void setup()
{
    Serial.begin(9600);
    screen.begin();
    ultrasonic.attach(TRIG_PIN, ECHO_PIN, MIN_DISTANCE, MAX_DISTANCE);
}

void updateZone(float height)
{
    // Only change zone when we clearly cross a boundary —
    // the gap between ENTER and EXIT prevents flickering.
    if (height < GROUND_LOW_EXIT)
    {
        currentZone = 1;
    }
    else if (height >= GROUND_LOW_ENTER && height < LOW_MID_EXIT)
    {
        currentZone = 2;
    }
    else if (height >= LOW_MID_ENTER && height < MID_HIGH_EXIT)
    {
        currentZone = 3;
    }
    else if (height >= MID_HIGH_ENTER)
    {
        currentZone = 4;
    }
    // If height is in a dead zone gap, currentZone stays unchanged
}

void loop()
{
    if (millis() - lastRead >= READ_INTERVAL)
    {
        lastRead = millis();
        heightAboveTable = ultrasonic.getDistanceCM();
        updateZone(heightAboveTable);
    }

    screen.beginDraw();
    screen.background(OFF);
    screen.stroke(ON);

    if (currentZone == 1)
    {
        // GROUND — single dot
        screen.point(6, 4);
    }
    else if (currentZone == 2)
    {
        // LOW — small circle
        screen.fill(ON);
        screen.circle(6, 4, 3);
    }
    else if (currentZone == 3)
    {
        // MID — large circle
        screen.fill(ON);
        screen.circle(6, 4, 6);
    }
    else
    {
        // HIGH — full cross
        screen.line(0, 4, 11, 4);
        screen.line(6, 0, 6, 7);
    }

    screen.endDraw();
}
```

**What changed:** Each boundary now has two values — an ENTER threshold and an EXIT threshold — creating a gap where the current zone holds steady. This is called [hysteresis](https://en.wikipedia.org/wiki/Hysteresis) and it's a common technique for preventing state flickering in sensor-driven interactions. If the sensor hovers at 25 cm, it won't jump back and forth between zones.

---

### Example: Threshold with Fill Pattern

Each zone fills a different **pattern** on the matrix. LOW shows a flat line at the bottom, MID fills the lower half, HIGH fills the entire screen. The visual feels like a liquid level rising as you lift.

```cpp
#include "TinyFilmFestival.h"
#include <EasyUltrasonic.h>

TinyScreen screen;
EasyUltrasonic ultrasonic;

int TRIG_PIN = A0;
int ECHO_PIN = A1;

int MIN_DISTANCE = 2;
int MAX_DISTANCE = 60;

int LOW_TO_MID = 15;
int MID_TO_HIGH = 35;

int READ_INTERVAL = 20;
unsigned long lastRead = 0;

float heightAboveTable = 0;
int currentZone = 1;

void setup()
{
    Serial.begin(9600);
    screen.begin();
    ultrasonic.attach(TRIG_PIN, ECHO_PIN, MIN_DISTANCE, MAX_DISTANCE);
}

void loop()
{
    if (millis() - lastRead >= READ_INTERVAL)
    {
        lastRead = millis();
        heightAboveTable = ultrasonic.getDistanceCM();

        if (heightAboveTable < LOW_TO_MID)
            currentZone = 1;
        else if (heightAboveTable < MID_TO_HIGH)
            currentZone = 2;
        else
            currentZone = 3;
    }

    screen.beginDraw();
    screen.background(OFF);
    screen.stroke(ON);

    if (currentZone == 1)
    {
        // LOW — bottom row only
        screen.line(0, 7, 11, 7);
    }
    else if (currentZone == 2)
    {
        // MID — bottom half filled
        screen.fill(ON);
        screen.rect(0, 4, 12, 4);
    }
    else
    {
        // HIGH — entire screen filled
        screen.fill(ON);
        screen.rect(0, 0, 12, 8);
    }

    screen.endDraw();
}
```

**What changed:** The shapes are designed as **stages of filling** rather than distinct symbols. This creates a visual metaphor — the screen "fills up" as the object rises, like a level gauge. The same threshold logic drives a completely different visual concept just by changing what's inside each zone block.

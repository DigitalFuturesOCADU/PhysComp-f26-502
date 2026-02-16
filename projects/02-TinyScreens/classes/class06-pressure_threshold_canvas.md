# Pressure Thresholds → Shapes: Step by Step

[← Back to Class 06](class06-Feb13.md)

This walkthrough takes you from a raw pressure reading to a threshold-based interaction in **five steps**. Instead of continuously mapping a sensor value to a single parameter (like the [Pressure → Screen](class06-pressure_map_canvas.md) walkthrough does), here you **divide the sensor range into zones** and draw a different shape in each zone.

The scenario: you're **squeezing a custom FSR** (force-sensing resistor made from Velostat and copper tape). The sensor measures how hard you're gripping, and the screen changes what it shows at different pressure levels.

If you've already done the [Distance Thresholds → Shapes](class06-distance_threshold_canvas.md) walkthrough, this follows the exact same structure — only the sensor and variable names change. No external library is needed for the pressure sensor — just `analogRead()`.

---

## Step 1 — Read the Sensor

**Goal:** Confirm the pressure sensor is wired correctly and see the raw values it produces.

Build your custom FSR and wire it with the voltage divider circuit as shown in the [Class 06 wiring diagram](class06-Feb13.md#pressure-sensor--how-does-it-work). Then upload this sketch and open **Tools → Serial Monitor** at 9600 baud.

```cpp
// ============================================================
// Step 1 — Read the pressure sensor and print to Serial Monitor
// ============================================================
// What this does:
//   Reads the custom FSR every 20 ms and prints the raw
//   analog value to the Serial Monitor.
//
// Why this matters:
//   You need to SEE the real numbers your sensor produces
//   before you can do anything useful with them.
//   analogRead() returns 0–1023, but YOUR sensor's usable
//   range will be narrower depending on how it's built.
// ============================================================

// --- Pin configuration ---
int FSR_PIN = A0;

// --- Timing ---
int READ_INTERVAL = 20;
unsigned long lastRead = 0;

// --- Sensor value ---
int gripStrength = 0;   // how hard you're squeezing (0–1023)

void setup()
{
    Serial.begin(9600);
}

void loop()
{
    if (millis() - lastRead >= READ_INTERVAL)
    {
        lastRead = millis();

        gripStrength = analogRead(FSR_PIN);

        Serial.print("Grip strength: ");
        Serial.println(gripStrength);
    }
}
```

### What to do with this

1. Upload the code and open the **Serial Monitor** (Tools → Serial Monitor, 9600 baud).
2. Press the sensor gently, then squeeze firmly.
3. **Write down three things:**
   - The value when you're **barely touching** the sensor — your minimum
   - The value when you're **squeezing as hard as you can** — your maximum
   - Roughly where the **middle** of that range falls

You'll use these numbers to define your threshold boundaries in Step 2.

---

## Step 2 — Define the Thresholds

**Goal:** Split the sensor range into three named zones and print which zone you're in.

Instead of using `map()` to create a continuous output, we use `if / else if / else` to divide the range into **discrete zones**. Each zone gets a descriptive name that matches the interaction: `LIGHT`, `MEDIUM`, `FIRM`.

```cpp
// ============================================================
// Step 2 — Divide the grip strength into three zones
// ============================================================
// What's new:
//   - Two threshold boundaries split the range into zones
//   - if / else if / else determines which zone you're in
//   - currentZone tracks the zone as a number (1, 2, 3)
//   - Serial Monitor prints both the reading and the zone name
//
// Why thresholds instead of map():
//   Sometimes you don't want a smooth gradient — you want
//   distinct states. "Resting," "pressing," and "squeezing
//   hard" are three different states, not a spectrum.
// ============================================================

// --- Pin configuration ---
int FSR_PIN = A0;

// --- Threshold boundaries (analog reading 0–1023) ---
// These divide the sensor range into three zones.
// Adjust these based on YOUR Serial Monitor observations from Step 1.
int LIGHT_TO_MEDIUM = 200;    // below this = LIGHT zone (barely pressing)
int MEDIUM_TO_FIRM = 600;     // above this = FIRM zone (squeezing hard)
                               // between = MEDIUM zone

// --- Timing ---
int READ_INTERVAL = 20;
unsigned long lastRead = 0;

// --- Values ---
int gripStrength = 0;
int currentZone = 0;       // 1 = LIGHT, 2 = MEDIUM, 3 = FIRM

void setup()
{
    Serial.begin(9600);
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
            currentZone = 1;  // LIGHT — barely pressing
        }
        else if (gripStrength < MEDIUM_TO_FIRM)
        {
            currentZone = 2;  // MEDIUM — pressing steadily
        }
        else
        {
            currentZone = 3;  // FIRM — squeezing hard
        }

        // Print both values so we can verify the zones
        Serial.print("Grip: ");
        Serial.print(gripStrength);
        Serial.print(" → Zone: ");
        Serial.println(currentZone);
    }
}
```

### What to notice

- **Two boundaries create three zones.** `LIGHT_TO_MEDIUM` and `MEDIUM_TO_FIRM` are all you need. Everything below the first is LIGHT, everything above the second is FIRM, everything between is MEDIUM.
- **The zone names describe the interaction.** Reading `LIGHT_TO_MEDIUM = 200` tells you that below 200 is considered a light touch. Compare that to `THRESHOLD_1 = 200` — same number, but no meaning.
- **`currentZone` is a simple number.** Using 1, 2, 3 makes it easy to use in `if` statements later. In Step 3 we'll use this number to choose which shape to draw.
- **Tuning:** If the Serial Monitor shows the zone jumping erratically, widen the gap between thresholds. If a zone never triggers, adjust the boundary values to match your actual sensor range.

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
//   - Three shapes: dot (LIGHT), circle (MEDIUM), cross (FIRM)
//   - if / else if / else picks the right shape based on zone
//   - currentZone is fixed at 2 — change it to 1 or 3 to
//     test each shape before connecting the sensor
//
// The sensor is still here and printing, but the shapes
// are NOT connected to it yet.
// ============================================================

#include "TinyFilmFestival.h"

TinyScreen screen;

// --- Pin configuration ---
int FSR_PIN = A0;

// --- Threshold boundaries ---
int LIGHT_TO_MEDIUM = 200;
int MEDIUM_TO_FIRM = 600;

// --- Timing ---
int READ_INTERVAL = 20;
unsigned long lastRead = 0;

// --- Values ---
int gripStrength = 0;

// --- Zone ---
// Fixed for now — change this to 1, 2, or 3 to test each shape
int currentZone = 2;

void setup()
{
    Serial.begin(9600);
    screen.begin();
}

void loop()
{
    // --- Sensor reading (still printing, not connected to shapes) ---
    if (millis() - lastRead >= READ_INTERVAL)
    {
        lastRead = millis();
        gripStrength = analogRead(FSR_PIN);

        Serial.print("Grip: ");
        Serial.print(gripStrength);
        Serial.print(" (zone fixed at ");
        Serial.print(currentZone);
        Serial.println(")");
    }

    // --- Draw the shape for the current zone ---
    screen.beginDraw();
    screen.background(OFF);
    screen.stroke(ON);

    if (currentZone == 1)
    {
        // LIGHT — small dot in the center (barely pressing)
        screen.point(6, 4);
    }
    else if (currentZone == 2)
    {
        // MEDIUM — filled circle (pressing steadily)
        screen.fill(ON);
        screen.circle(6, 4, 5);
    }
    else
    {
        // FIRM — cross pattern (squeezing hard)
        screen.line(0, 4, 11, 4);  // horizontal line
        screen.line(6, 0, 6, 7);   // vertical line
    }

    screen.endDraw();
}
```

### What to notice

- The `if / else if / else` block inside `loop()` picks which shape to draw based on `currentZone`. This is the same pattern from Step 2, but now it controls drawing instead of printing.
- **Change `currentZone` to 1, 2, or 3**, upload, and confirm each shape looks right before connecting the sensor. This is the static-to-dynamic pattern in action.
- The three shapes have increasing visual weight: a single dot → a filled circle → a full cross. This creates a clear visual progression as you squeeze harder.
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

TinyScreen screen;

// --- Pin configuration ---
int FSR_PIN = A0;

// --- Threshold boundaries ---
int LIGHT_TO_MEDIUM = 200;
int MEDIUM_TO_FIRM = 600;

// --- Timing ---
int READ_INTERVAL = 20;
unsigned long lastRead = 0;

// --- Values ---
int gripStrength = 0;

// --- Zone ---
// Still fixed for testing — we'll connect the sensor in Step 5
int currentZone = 2;

void setup()
{
    Serial.begin(9600);
    screen.begin();
}

// --- Draw the shape for a given zone ---
// This is the same if / else if / else from Step 3,
// pulled out into its own function.
void drawShape(int zone)
{
    if (zone == 1)
    {
        // LIGHT — small dot in the center (barely pressing)
        screen.point(6, 4);
    }
    else if (zone == 2)
    {
        // MEDIUM — filled circle (pressing steadily)
        screen.fill(ON);
        screen.circle(6, 4, 5);
    }
    else
    {
        // FIRM — cross pattern (squeezing hard)
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
        gripStrength = analogRead(FSR_PIN);

        Serial.print("Grip: ");
        Serial.print(gripStrength);
        Serial.print(" (zone fixed at ");
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

**Goal:** Replace the fixed zone number with the threshold logic from Step 2 so the shape on screen changes as you squeeze harder.

One section changes: `currentZone` is now computed from the sensor value using `if / else if / else`, instead of being fixed at 2.

```cpp
// ============================================================
// Step 5 — Grip strength controls which shape is drawn
// ============================================================
// What changed:
//   currentZone is now computed from gripStrength using
//   the threshold boundaries. The drawShape() function from
//   Step 4 handles the rest — squeeze to see the shapes change.
// ============================================================

#include "TinyFilmFestival.h"

TinyScreen screen;

// --- Pin configuration ---
int FSR_PIN = A0;

// --- Threshold boundaries (from YOUR observations) ---
int LIGHT_TO_MEDIUM = 200;    // below = barely pressing
int MEDIUM_TO_FIRM = 600;     // above = squeezing hard

// --- Timing ---
int READ_INTERVAL = 20;
unsigned long lastRead = 0;

// --- Values ---
int gripStrength = 0;
int currentZone = 1;       // now driven by the sensor

void setup()
{
    Serial.begin(9600);
    screen.begin();
}

// --- Draw the shape for a given zone ---
void drawShape(int zone)
{
    if (zone == 1)
    {
        // LIGHT — small dot (barely pressing)
        screen.point(6, 4);
    }
    else if (zone == 2)
    {
        // MEDIUM — filled circle (pressing steadily)
        screen.fill(ON);
        screen.circle(6, 4, 5);
    }
    else
    {
        // FIRM — cross pattern (squeezing hard)
        screen.line(0, 4, 11, 4);
        screen.line(6, 0, 6, 7);
    }
}

void loop()
{
    if (millis() - lastRead >= READ_INTERVAL)
    {
        lastRead = millis();
        gripStrength = analogRead(FSR_PIN);

        // ★ THE CONNECTION — grip strength determines the zone
        if (gripStrength < LIGHT_TO_MEDIUM)
        {
            currentZone = 1;  // LIGHT
        }
        else if (gripStrength < MEDIUM_TO_FIRM)
        {
            currentZone = 2;  // MEDIUM
        }
        else
        {
            currentZone = 3;  // FIRM
        }

        Serial.print("Grip: ");
        Serial.print(gripStrength);
        Serial.print(" → Zone ");
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
| **Zone boundaries** | `LIGHT_TO_MEDIUM` / `MEDIUM_TO_FIRM` | Shift where the transitions happen. Closer values = smaller MEDIUM zone. |
| **Shapes** | Inside `drawShape()` | Swap in any shapes you want — `screen.rect()`, `screen.circle()`, `screen.line()`, `screen.point()`. |
| **Number of zones** | Add more `else if` blocks | You can have 4, 5, or more zones — just add more threshold boundaries and shapes. |
| **Dead zone** | Add a gap between thresholds | If the display flickers at a boundary, leave a small gap (e.g., LIGHT < 180, MEDIUM > 220) — values in the gap keep the previous zone. |

### What's Next?

You now have the core pattern: **read → classify → draw**. This is different from the `map()` pattern — instead of smooth, continuous output, you get distinct **states**. From here you can:

- Add **continuous mapping within each zone** — see Going Further below
- Control **animations per zone** instead of shapes — see [Distance Thresholds → Animation](class06-distance_threshold_animation.md)
- Try the same threshold pattern with a **distance sensor** — see [Distance Thresholds → Shapes](class06-distance_threshold_canvas.md)

---

## Going Further

The examples below all start from the Step 5 code. Each one extends the threshold idea in a different direction. Comments highlight what's different.

---

### Example: Size Changes Within Each Zone

Thresholds pick the shape, but `map()` controls the **size within each zone**. The shape changes at the boundary, and within each zone the shape grows as you squeeze harder.

```cpp
#include "TinyFilmFestival.h"

TinyScreen screen;

int FSR_PIN = A0;

// --- Threshold boundaries ---
int LIGHT_TO_MEDIUM = 200;
int MEDIUM_TO_FIRM = 600;

int READ_INTERVAL = 20;
unsigned long lastRead = 0;

int gripStrength = 0;
int currentZone = 1;
int shapeSize = 1;         // size parameter within each zone

void setup()
{
    Serial.begin(9600);
    screen.begin();
}

void drawShape(int zone, int size)
{
    if (zone == 1)
    {
        // LIGHT — dot cluster, size controls how many dots
        int dots = constrain(size, 1, 4);
        screen.point(6, 4);
        if (dots >= 2) screen.point(5, 4);
        if (dots >= 3) screen.point(7, 4);
        if (dots >= 4) { screen.point(6, 3); screen.point(6, 5); }
    }
    else if (zone == 2)
    {
        // MEDIUM — circle, size controls diameter
        screen.fill(ON);
        screen.circle(6, 4, size);
    }
    else
    {
        // FIRM — cross, size controls arm length
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
        gripStrength = analogRead(FSR_PIN);

        if (gripStrength < LIGHT_TO_MEDIUM)
        {
            currentZone = 1;
            // Map within the LIGHT zone: 0–200 → size 1–4
            shapeSize = map(gripStrength, 0, LIGHT_TO_MEDIUM, 1, 4);
            shapeSize = constrain(shapeSize, 1, 4);
        }
        else if (gripStrength < MEDIUM_TO_FIRM)
        {
            currentZone = 2;
            // Map within the MEDIUM zone: 200–600 → diameter 2–8
            shapeSize = map(gripStrength, LIGHT_TO_MEDIUM, MEDIUM_TO_FIRM, 2, 8);
            shapeSize = constrain(shapeSize, 2, 8);
        }
        else
        {
            currentZone = 3;
            // Map within the FIRM zone: 600–1023 → arm length 4–12
            shapeSize = map(gripStrength, MEDIUM_TO_FIRM, 1023, 4, 12);
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

Each zone's shape **oscillates** at a different speed. The LIGHT zone barely moves, MEDIUM bounces gently, and FIRM bounces fast — the harder you squeeze, the more energetic the motion.

```cpp
#include "TinyFilmFestival.h"

TinyScreen screen;

int FSR_PIN = A0;

int LIGHT_TO_MEDIUM = 200;
int MEDIUM_TO_FIRM = 600;

// --- Bounce speed per zone (ms per cycle) ---
int LIGHT_BOUNCE = 4000;     // very slow drift
int MEDIUM_BOUNCE = 1500;    // gentle bounce
int FIRM_BOUNCE = 400;       // fast bounce

int READ_INTERVAL = 20;
unsigned long lastRead = 0;

int gripStrength = 0;
int currentZone = 1;
int bounceSpeed = 4000;

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

        if (gripStrength < LIGHT_TO_MEDIUM)
        {
            currentZone = 1;
            bounceSpeed = LIGHT_BOUNCE;
        }
        else if (gripStrength < MEDIUM_TO_FIRM)
        {
            currentZone = 2;
            bounceSpeed = MEDIUM_BOUNCE;
        }
        else
        {
            currentZone = 3;
            bounceSpeed = FIRM_BOUNCE;
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

**What changed:** Each zone now has a fixed `bounceSpeed` that gets passed to `oscillateInt()`. The shape still changes at the thresholds, but within each zone the shape bounces vertically. The bounce speed jumps discretely at each boundary — LIGHT drifts slowly, MEDIUM bounces, FIRM jitters.

---

### Example: Four Zones with Dead Zones

Add a **fourth zone** and introduce **dead zones** — small gaps between thresholds where the display keeps its current state. This prevents flickering when the sensor value hovers near a boundary.

```cpp
#include "TinyFilmFestival.h"

TinyScreen screen;

int FSR_PIN = A0;

// --- Threshold boundaries with dead zones ---
// Zone 1 (REST):    below 100
// Dead zone:        100–130  (keeps current zone)
// Zone 2 (LIGHT):   130–350
// Dead zone:        350–380  (keeps current zone)
// Zone 3 (MEDIUM):  380–650
// Dead zone:        650–680  (keeps current zone)
// Zone 4 (FIRM):    above 680

int REST_LIGHT_ENTER = 130;     // must rise above 130 to enter LIGHT
int REST_LIGHT_EXIT = 100;      // must drop below 100 to return to REST
int LIGHT_MEDIUM_ENTER = 380;
int LIGHT_MEDIUM_EXIT = 350;
int MEDIUM_FIRM_ENTER = 680;
int MEDIUM_FIRM_EXIT = 650;

int READ_INTERVAL = 20;
unsigned long lastRead = 0;

int gripStrength = 0;
int currentZone = 1;

void setup()
{
    Serial.begin(9600);
    screen.begin();
}

void updateZone(int pressure)
{
    // Only change zone when we clearly cross a boundary —
    // the gap between ENTER and EXIT prevents flickering.
    if (pressure < REST_LIGHT_EXIT)
    {
        currentZone = 1;
    }
    else if (pressure >= REST_LIGHT_ENTER && pressure < LIGHT_MEDIUM_EXIT)
    {
        currentZone = 2;
    }
    else if (pressure >= LIGHT_MEDIUM_ENTER && pressure < MEDIUM_FIRM_EXIT)
    {
        currentZone = 3;
    }
    else if (pressure >= MEDIUM_FIRM_ENTER)
    {
        currentZone = 4;
    }
    // If pressure is in a dead zone gap, currentZone stays unchanged
}

void loop()
{
    if (millis() - lastRead >= READ_INTERVAL)
    {
        lastRead = millis();
        gripStrength = analogRead(FSR_PIN);
        updateZone(gripStrength);
    }

    screen.beginDraw();
    screen.background(OFF);
    screen.stroke(ON);

    if (currentZone == 1)
    {
        // REST — single dot
        screen.point(6, 4);
    }
    else if (currentZone == 2)
    {
        // LIGHT — small circle
        screen.fill(ON);
        screen.circle(6, 4, 3);
    }
    else if (currentZone == 3)
    {
        // MEDIUM — large circle
        screen.fill(ON);
        screen.circle(6, 4, 6);
    }
    else
    {
        // FIRM — full cross
        screen.line(0, 4, 11, 4);
        screen.line(6, 0, 6, 7);
    }

    screen.endDraw();
}
```

**What changed:** Each boundary now has two values — an ENTER threshold and an EXIT threshold — creating a gap where the current zone holds steady. This is called [hysteresis](https://en.wikipedia.org/wiki/Hysteresis) and it's a common technique for preventing state flickering in sensor-driven interactions. If the sensor hovers at 350, it won't jump back and forth between zones.

---

### Example: Threshold with Fill Pattern

Each zone fills a different **pattern** on the matrix. LIGHT shows a flat line at the bottom, MEDIUM fills the lower half, FIRM fills the entire screen. The visual feels like a pressure gauge filling up as you squeeze.

```cpp
#include "TinyFilmFestival.h"

TinyScreen screen;

int FSR_PIN = A0;

int LIGHT_TO_MEDIUM = 200;
int MEDIUM_TO_FIRM = 600;

int READ_INTERVAL = 20;
unsigned long lastRead = 0;

int gripStrength = 0;
int currentZone = 1;

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

        if (gripStrength < LIGHT_TO_MEDIUM)
            currentZone = 1;
        else if (gripStrength < MEDIUM_TO_FIRM)
            currentZone = 2;
        else
            currentZone = 3;
    }

    screen.beginDraw();
    screen.background(OFF);
    screen.stroke(ON);

    if (currentZone == 1)
    {
        // LIGHT — bottom row only
        screen.line(0, 7, 11, 7);
    }
    else if (currentZone == 2)
    {
        // MEDIUM — bottom half filled
        screen.fill(ON);
        screen.rect(0, 4, 12, 4);
    }
    else
    {
        // FIRM — entire screen filled
        screen.fill(ON);
        screen.rect(0, 0, 12, 8);
    }

    screen.endDraw();
}
```

**What changed:** The shapes are designed as **stages of filling** rather than distinct symbols. This creates a visual metaphor — the screen "fills up" as you squeeze harder, like a pressure gauge. The same threshold logic drives a completely different visual concept just by changing what's inside each zone block.

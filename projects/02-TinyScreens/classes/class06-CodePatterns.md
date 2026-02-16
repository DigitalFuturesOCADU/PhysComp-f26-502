# Class 06 — Code Patterns Reference

These sections support the [Class 06 Workshop](class06-Feb13.md). Read through them before starting the sensor examples — the patterns here apply to every example on that page.

---

## Planning Your Code

Before writing code, plan in plain language. Think of your sketch as having three layers: **read** (get the sensor value), **translate** (decide what it means), and **draw** (update the screen). Naming things well at each layer makes everything easier.

### Input Language

When writing code, refer to data by **what it actually is** in your system rather than generic numbers. Good variable names describe the data's meaning:

**Distance Examples:**

```cpp
// ❌ Generic — what does "val" mean?
int val = ultrasonic.getDistanceCM();

// ✅ Descriptive — names say what the data IS
int distanceToHand = ultrasonic.getDistanceCM();    // "toward" concept
int proximityLevel = ultrasonic.getDistanceCM();    // "above" concept
int gapBetween = ultrasonic.getDistanceCM();        // "between" concept
```

**Pressure Examples:**

```cpp
// ❌ Generic
int reading = analogRead(A0);

// ✅ Descriptive
int gripStrength = analogRead(A0);     // "against" concept
int squeezeForce = analogRead(A0);     // "into" concept
int weightOnSurface = analogRead(A0);  // "upon" concept
```

### Output Language

The same approach applies to output. Describe what the screen *should do* before worrying about exact code:

```
Plain Language:                          Code:
─────────────────────────────────────    ──────────────────────────────────
"The circle grows as pressure            int diameter = map(squeezeForce,
 increases"                                  0, 1023, 1, 7);
                                         screen.circle(5, 3, diameter);

"The circle moves toward the edge         int circleX = map(distanceToHand,
 as I get closer"                            5, 100, 11, 0);
                                         screen.circle(circleX, 4, 3);
```

> **Tip:** Comments are free. Use them as your plain-language plan right in the code. Future-you will thank present-you.

### Parameters — The Bridge Between Input and Output

A **parameter** is any value in your output code that *could* change based on input. When you identify parameters, you find the connection points between sensors and the screen:

| Output Description | Fixed Code | Parameter | What Could Drive It |
|---|---|---|---|
| A circle at position x | `screen.circle(5, 4, 3)` | `5` (x position) | Distance sensor |
| A circle with diameter d | `screen.circle(5, 3, 4)` | `4` (diameter) | Pressure sensor |
| Animation at speed s | `screen.setSpeed(1.0)` | `1.0` (speed) | Either sensor |
| Rectangle with width w | `screen.rect(0, 0, 8, 8)` | `8` (width) | Either sensor |

### The Loop Pattern

Every pass through `loop()` does the same thing: **read** the sensor, **translate** the value, **draw** to the screen. There are two ways to translate:

**`map()` — Continuous:** The sensor smoothly controls a visual parameter (position, size, speed):

```cpp
void loop()
{
    // READ
    int distance = ultrasonic.getDistanceCM();

    // TRANSLATE — map sensor range to screen range
    int circleX = map(distance, 2, 100, 11, 0);
    circleX = constrain(circleX, 0, 11);

    // DRAW
    screen.beginDraw();
    screen.background(OFF);
    screen.stroke(ON);
    screen.fill(ON);
    screen.circle(circleX, 4, 3);
    screen.endDraw();
}
```

**`if/else` — Threshold:** The sensor switches between distinct behaviors at a boundary:

```cpp
void loop()
{
    // READ
    int distance = ultrasonic.getDistanceCM();

    // TRANSLATE + DRAW — check threshold and respond
    if (distance < CLOSE_THRESHOLD)
    {
        screen.play(alert, LOOP);
    }
    else
    {
        screen.play(calm, LOOP);
    }

    screen.update();
}
```

#### Name Your Thresholds

Use **named variables** instead of raw numbers so your code explains the decision:

```cpp
// ❌ What does 15 mean? Hard to understand.
if (distanceToHand < 15)
{
    ...
}

// ✅ The variable name explains the decision
int personalSpace = 15;  // cm — closer than this feels "too close"
if (distanceToHand < personalSpace)
{
    ...
}
```

The same idea works for pressure:

```cpp
// ❌ Magic number
if (gripStrength > 200)
{
    ...
}

// ✅ Clear intent
int firmGrip = 200;  // analog reading when user grips firmly
if (gripStrength > firmGrip)
{
    ...
}
```

More thresholds create more zones. One threshold = two zones; two thresholds = three zones:

```
Distance:  0 cm ──────── 15 cm ──────── 40 cm ──────── 100+ cm
Zone:       [  CLOSE  ]   [  NEAR  ]     [   FAR    ]
```

---

## Writing Functions

A **function** is a named block of code that performs a specific task. You've already been using them — `setup()`, `loop()`, `analogRead()`, and `map()` are all functions. Now you'll write your own.

### Why Use Functions?

- **Organize** — Each step of your logic gets its own clearly labeled block
- **Reuse** — Write it once, call it anywhere
- **Readability** — `loop()` reads like a plain-language plan

### Structure of a Function

```cpp
returnType functionName(parameterType parameterName)
{
    // Code that does the work
    return value;  // Send a result back (if needed)
}
```

### Key Words

| Word | What it means |
|------|---------------|
| **Return type** | What kind of data the function sends back: `int`, `float`, `String`, `bool`, or `void` (nothing) |
| **Parameters** | Values you pass *into* the function — listed inside the parentheses |
| **`return`** | Sends a value back to wherever the function was called and exits the function |
| **Call** | Using the function by writing its name with parentheses: `readDistance()` |

### Types of Functions

| Type | Example | Use when... |
|------|---------|-------------|
| **Returns a value** | `int distanceToX(int dist)` | You need a calculation back |
| **Does something (void)** | `void drawCircle(int x)` | You want to trigger an action, no result needed |
| **No parameters** | `int readDistance()` | The function gets its own data internally |
| **Multiple parameters** | `int clampValue(int val, int lo, int hi)` | You need to pass in several pieces of info |

### Example: Functions for Each Step

```cpp
// --- Sensor range ---
int MIN_DISTANCE = 2;
int MAX_DISTANCE = 100;

// --- Screen bounds ---
int SCREEN_MIN_X = 0;
int SCREEN_MAX_X = 11;

// ── STEP 1 FUNCTION: Get new data ──
// Returns an int, takes no parameters
int readDistance()
{
    return ultrasonic.getDistanceCM();
}

// ── STEP 2 FUNCTION: Map to output ──
// Returns an int, takes an int parameter
int distanceToX(int distance)
{
    int x = map(distance, MIN_DISTANCE, MAX_DISTANCE, SCREEN_MAX_X, SCREEN_MIN_X);
    return constrain(x, SCREEN_MIN_X, SCREEN_MAX_X);
}

// ── STEP 3 FUNCTION: Draw ──
// Returns nothing (void), takes an int parameter
void drawCircle(int x)
{
    screen.beginDraw();
    screen.background(OFF);
    screen.stroke(ON);
    screen.fill(ON);
    screen.circle(x, 4, 3);
    screen.endDraw();
}

// ── loop() reads like a plan ──
void loop()
{
    int distance = readDistance();       // Step 1: Get data
    int circleX = distanceToX(distance); // Step 2: Map to output
    drawCircle(circleX);                 // Step 3: Draw
}
```

> **Tip:** Notice how `loop()` now reads almost like plain English. Each function name describes *what* happens; the function body describes *how*.

---

## From Static to Dynamic

The key shift in this class is going from **fixed values** (hard-coded numbers) to **dynamic values** (driven by sensor input). Here's how to think through that transformation:

### Step 1: Start with a Class 05 Sketch

In Class 05, you wrote an [Animated Circle](class05-Feb06.md#example-animated-circle-with-expanding-diameter) that used `oscillateInt()` to grow and shrink the diameter on a timer:

```cpp
// Class 05 — Animated Circle with Expanding Diameter
int diameter = oscillateInt(1, 7, 2000); // Grows and shrinks over 2 seconds

screen.beginDraw();
screen.background(OFF);
screen.stroke(ON);
screen.noFill();
screen.circle(5, 3, diameter);
screen.endDraw();
```

### Step 2: Identify the Parameter

Which value could a sensor control instead of `oscillateInt()`?

```cpp
//                  ↓ this one — diameter
int diameter = oscillateInt(1, 7, 2000);
screen.circle(5, 3, diameter);
```

### Step 3: Replace with a Named Variable

Pull it out so it's easy to swap the source:

```cpp
int diameter = 4;   // fixed for now
screen.circle(5, 3, diameter);
```

### Step 4: Drive the Variable with Sensor Data

Replace the fixed value with a `map()` call that reads from a sensor:

```cpp
int pressure = analogRead(FSR_PIN);
int diameter = map(pressure, 50, 800, 1, 7);
diameter = constrain(diameter, 1, 7);
screen.circle(5, 3, diameter);
```

The circle that used to grow and shrink on a timer now grows and shrinks based on how hard you press.

### The Same Pattern Works Everywhere

Each Class 06 example below follows this same transformation — find the parameter, replace its source with sensor data:

| Class 05 (time-driven) | Class 06 (sensor-driven) | Example below |
|---|---|---|
| `oscillateInt(0, 11, 2000)` for circle position | `map(distance, ...)` for circle position | [Distance-Controlled Circle](class06-Feb13.md#example-distance-mapped-to-canvas) |
| `oscillateInt(1, 7, 2000)` for circle diameter | `map(pressure, ...)` for circle diameter | [Pressure-Controlled Circle](class06-Feb13.md#example-pressure-mapped-to-canvas) |
| `screen.setSpeed(1.0)` fixed speed | `map(sensor, ...)` for dynamic speed | [Distance Controls Speed](class06-Feb13.md#example-distance-controls-animation-speed) / [Pressure Controls Speed](class06-Feb13.md#example-pressure-controls-animation-speed) |
| `screen.play(anim, LOOP)` one animation | Threshold `if/else` picks which animation | [Distance Threshold](class06-Feb13.md#example-distance-threshold--switch-animations) / [Pressure Threshold](class06-Feb13.md#example-pressure-threshold--switch-animations) |

### Combining Sensor Data with Library Animation Tools

You don't have to choose one or the other. In Class 05, [Two Dots Moving in Opposite Directions](class05-Feb06.md#example-two-dots-moving-in-opposite-directions) used two `oscillateInt()` calls. Here, the distance sensor replaces one axis while `oscillateInt()` still handles the other:

```cpp
void loop()
{
    float distance = ultrasonic.getDistanceCM();

    // Sensor controls horizontal position
    int xPos = map(distance, 2, 100, 0, 11);
    xPos = constrain(xPos, 0, 11);

    // oscillateInt controls vertical bounce — independent of sensor
    int yPos = oscillateInt(1, 6, 1000);

    screen.beginDraw();
    screen.background(OFF);
    screen.stroke(ON);
    screen.fill(ON);
    screen.circle(xPos, yPos, 3);
    screen.endDraw();
}
```

# Class 05 | February 6 | Workshop 1

[← Back to Tiny Screens](../TinyScreens.md)

## Overview

This is the first workshop for the [Tiny Screens](../TinyScreens.md) project. Over the coming weeks you will build an interactive object driven by sensor input and the Arduino's built-in 12×8 LED matrix. Before you can make it interactive, you need to know how to put things on the screen.

Today is focused on **output only** — learning the three ways to draw on the LED matrix using the TinyFilmFestival library. Sensors and interactivity come next class.

### What We'll Cover

1. [Getting Started](#getting-started) — install the library and understand the LED grid
2. [Key Concepts](#key-concepts) — how drawing methods relate, how timing works, and the `for` loop
3. [Method 1 — Simple LED](#method-1--simple-led-mode) — turn individual LEDs on and off
4. [Method 2 — Animation Mode](#method-2--animation-mode) — play pre-made frame animations from the LED Matrix Editor
5. [Method 3 — Canvas Mode](#method-3--canvas-mode) — draw shapes, text, and motion with code
6. [Workshop](#putting-it-all-together) — what to do for today's submission

## Lecture Slides

[Slides](https://ocaduniversity-my.sharepoint.com/:p:/g/personal/npuckett_ocadu_ca/IQDR3QsxAATQQYD733jdYSKqAUY6zCJwE8hp2ELeaFpUCkk?e=dfozRo)

## Resources

- [TinyFilmFestival Documentation](https://digitalfuturesocadu.github.io/TinyFilmFestival/docs/#home)
- [LED Matrix Editor](https://ledmatrix-editor.arduino.cc/)
- [Delay vs Millis](../../01-AltController/DelayVsMillis.md)

---

## Getting Started

### Installation

1. Open Arduino IDE → **Sketch → Include Library → Manage Libraries...**
2. Search for **"TinyFilmFestival"**
3. Click **Install** → Choose **"Install all"** when prompted for dependencies

> **Important:** Select "Install all" so that ArduinoGraphics and other required libraries are installed automatically.

### LED Matrix Layout

The Arduino UNO R4 WiFi has a built-in 12×8 LED matrix (96 LEDs total). Each LED can be addressed using `(x, y)` coordinates or a linear index (0–95).

```
     x=0  x=1  x=2  x=3  x=4  x=5  x=6  x=7  x=8  x=9  x=10 x=11
    +----+----+----+----+----+----+----+----+----+----+----+----+
y=0 |  0 |  1 |  2 |  3 |  4 |  5 |  6 |  7 |  8 |  9 | 10 | 11 |
    +----+----+----+----+----+----+----+----+----+----+----+----+
y=1 | 12 | 13 | 14 | 15 | 16 | 17 | 18 | 19 | 20 | 21 | 22 | 23 |
    +----+----+----+----+----+----+----+----+----+----+----+----+
y=2 | 24 | 25 | 26 | 27 | 28 | 29 | 30 | 31 | 32 | 33 | 34 | 35 |
    +----+----+----+----+----+----+----+----+----+----+----+----+
y=3 | 36 | 37 | 38 | 39 | 40 | 41 | 42 | 43 | 44 | 45 | 46 | 47 |
    +----+----+----+----+----+----+----+----+----+----+----+----+
y=4 | 48 | 49 | 50 | 51 | 52 | 53 | 54 | 55 | 56 | 57 | 58 | 59 |
    +----+----+----+----+----+----+----+----+----+----+----+----+
y=5 | 60 | 61 | 62 | 63 | 64 | 65 | 66 | 67 | 68 | 69 | 70 | 71 |
    +----+----+----+----+----+----+----+----+----+----+----+----+
y=6 | 72 | 73 | 74 | 75 | 76 | 77 | 78 | 79 | 80 | 81 | 82 | 83 |
    +----+----+----+----+----+----+----+----+----+----+----+----+
y=7 | 84 | 85 | 86 | 87 | 88 | 89 | 90 | 91 | 92 | 93 | 94 | 95 |
    +----+----+----+----+----+----+----+----+----+----+----+----+
```

- **(x, y)** — `x` is the column (0–11), `y` is the row (0–7)
- **Linear index** — the number inside each cell (0–95), used with `ledWrite(index, state)`

---

## Key Concepts

### The 3 Drawing Methods

The TinyFilmFestival library provides three ways to draw on the LED matrix. Each method works differently and is suited to different use cases.

| Method | What it does | Best for |
|--------|-------------|----------|
| **Simple LED** | Control individual LEDs like `digitalWrite()` | Turning specific LEDs on/off, basic patterns |
| **Animation Mode** | Play pre-made frame-by-frame animations | Pre-designed visuals, characters, icons |
| **Canvas Mode** | Draw shapes and text with code in real-time | Dynamic graphics, moving shapes, sensor-driven visuals |

> **Note:** Simple LED mode uses its own internal buffer — don't mix `ledWrite()` with Animation or Canvas mode in the same sketch. Animation and Canvas modes can be combined.

---

### Thinking in Frames

When working with the LED matrix, it helps to think of your `loop()` function as a **frame renderer**. Every time `loop()` runs, it draws one frame of your animation. The Arduino calls `loop()` over and over as fast as it can — each call is a chance to decide what the screen looks like *right now*.

```
Frame 1          Frame 2          Frame 3          Frame 4
┌────────────┐   ┌────────────┐   ┌────────────┐   ┌────────────┐
│ ●          │   │   ●        │   │      ●     │   │         ●  │
│            │   │            │   │            │   │            │
│            │   │            │   │            │   │            │
└────────────┘   └────────────┘   └────────────┘   └────────────┘
  loop() #1        loop() #2        loop() #3        loop() #4
     ↑                ↑                ↑                ↑
  millis()=0     millis()=100    millis()=200    millis()=300
```

Each frame, you control **what changes** and **what stays the same** since the last one. A dot that moves? Change its `x` position. A background pattern that persists? Draw it the same way every frame.

### millis() Is Your Shared Clock

`millis()` returns the number of milliseconds since the Arduino powered on. It keeps counting no matter what, and every part of your code can read it. This makes it the perfect reference point for coordinating time across your sketch:

- **"Has enough time passed to move the dot?"** → Compare `millis()` to a stored timestamp
- **"How far along is this animation cycle?"** → `oscillateInt()` uses `millis()` internally to figure this out for you
- **"When should I switch states?"** → Store `millis()` when you enter a state, check elapsed time each frame

```
millis():  0ms     100ms    200ms    300ms    400ms    500ms
           │        │        │        │        │        │
loop():    ■────────■────────■────────■────────■────────■──── ...
           │                          │
           └── "not yet"              └── "300ms passed — time to move!"
```

The key rule: **never use `delay()` in `loop()`**. `delay()` freezes the entire Arduino — it can't read buttons, update animations, or respond to sensors while waiting. `millis()` lets you check time *without stopping*, so your code stays responsive and interactive. See [Delay vs Millis](../../01-AltController/DelayVsMillis.md) for a full explanation.

> The TinyFilmFestival library handles frame timing for you in Animation Mode (`screen.update()`) and provides `oscillateInt()` / `Ease` for Canvas Mode — both use `millis()` under the hood so you don't have to manage timing manually in most cases.

### The `for` Loop

A `for` loop repeats a block of code a set number of times. It's perfect for working with the LED matrix because you often want to do the same thing across a row or column of LEDs without writing out every single call by hand.

For example, to light up the entire top row **without** a `for` loop, you'd write:

```cpp
ledWrite(0, 0, HIGH);
ledWrite(1, 0, HIGH);
ledWrite(2, 0, HIGH);
ledWrite(3, 0, HIGH);
ledWrite(4, 0, HIGH);
ledWrite(5, 0, HIGH);
ledWrite(6, 0, HIGH);
ledWrite(7, 0, HIGH);
ledWrite(8, 0, HIGH);
ledWrite(9, 0, HIGH);
ledWrite(10, 0, HIGH);
ledWrite(11, 0, HIGH);
```

With a `for` loop, the same result in 4 lines:

```cpp
for (int x = 0; x < 12; x++)
{
    ledWrite(x, 0, HIGH); // x changes each pass: 0, 1, 2, ... 11
}
```

The three parts inside the parentheses are:

| Part | What it does | Example |
|------|-------------|--------|
| **Start** | Create a counter variable and set its initial value | `int x = 0` |
| **Condition** | Keep looping as long as this is true | `x < 12` |
| **Step** | Change the counter after each pass | `x++` (add 1) |

> A `for` loop runs **instantly** — all 12 passes happen within the same frame. It doesn't add any delay. See the [full for loop documentation](https://docs.arduino.cc/language-reference/en/structure/control-structure/for/) for more details.

---

## Method 1 — Simple LED Mode

Simple LED mode lets you control individual LEDs the same way you would use `digitalWrite()`. It's the most basic way to use the matrix and a great starting point.

> [Full Simple LED Documentation](https://digitalfuturesocadu.github.io/TinyFilmFestival/docs/#simple-led)

### Key Functions

| Function | Description |
|----------|-------------|
| `ledBegin()` | Initialize the matrix (call once in `setup()`) |
| `ledWrite(x, y, state)` | Set an LED at (x, y) to `HIGH` or `LOW` |
| `ledWrite(index, state)` | Set an LED by linear index (0–95) |
| `ledRead(x, y)` | Read the current state of an LED |
| `ledToggle(x, y)` | Toggle an LED on/off |
| `ledClear()` | Turn off all LEDs |
| `ledBlink(x, y, rateMs)` | Blink an LED at its own independent rate |
| `ledUpdate()` | Process blink timers (required in `loop()` if using blink) |

### Example: Light up the Four Corners

```cpp
#include "TinyFilmFestival.h"

void setup()
{
    ledBegin();
    
    // Turn on the four corner LEDs
    ledWrite(0, 0, HIGH);     // Top-left
    ledWrite(11, 0, HIGH);    // Top-right
    ledWrite(0, 7, HIGH);     // Bottom-left
    ledWrite(11, 7, HIGH);    // Bottom-right
}

void loop()
{
    // Nothing needed here — LEDs stay on
}
```

### Example: Blinking Center with Static Corners

```cpp
#include "TinyFilmFestival.h"

int blinkSpeed = 500; // Blink every 500ms

void setup()
{
    ledBegin();
    
    // Static corner LEDs
    ledWrite(0, 0, HIGH);
    ledWrite(11, 0, HIGH);
    ledWrite(0, 7, HIGH);
    ledWrite(11, 7, HIGH);
    
    // Blinking LEDs in the center (2x2 block)
    ledBlink(5, 3, blinkSpeed);
    ledBlink(6, 3, blinkSpeed);
    ledBlink(5, 4, blinkSpeed);
    ledBlink(6, 4, blinkSpeed);
}

void loop()
{
    ledUpdate(); // Required to drive blink timers
}
```

### Example: Sequentially Light a Row

This example turns on each LED in the top row one at a time, then clears and repeats. It uses `millis()` instead of `delay()` so the Arduino stays responsive.

```cpp
#include "TinyFilmFestival.h"

int currentLED = 0;
unsigned long previousMillis = 0;
const long stepInterval = 200;    // Time between each LED turning on
const long clearDelay = 500;      // Pause before clearing
bool clearing = false;

void setup()
{
    ledBegin();
}

void loop()
{
    unsigned long currentMillis = millis();

    if (!clearing)
    {
        // Turn on LEDs one at a time
        if (currentMillis - previousMillis >= stepInterval)
        {
            previousMillis = currentMillis;
            ledWrite(currentLED, 0, HIGH);
            currentLED++;

            if (currentLED >= 12)
            {
                clearing = true; // All LEDs on — start clear phase
            }
        }
    }
    else
    {
        // Wait, then clear and restart
        if (currentMillis - previousMillis >= clearDelay)
        {
            previousMillis = currentMillis;
            ledClear();
            currentLED = 0;
            clearing = false;
        }
    }
}
```

> **Coding Concept — State Machine:** This example introduces a simple [state machine](../TinyScreens.md#state-machines), one of the key coding concepts for this project. The `clearing` variable acts as a two-state switch: when `false`, the sketch is in the **"lighting"** state (turning on LEDs one at a time); when `true`, it enters the **"clearing"** state (waiting, then resetting). Each state has its own behavior and its own transition condition to move to the other state. In later classes, you'll expand this pattern to manage more complex phases of interaction — like idle, sensing, and responding — each with their own animations and transition logic.

### Example: Draw a Plus Sign

This example uses two `for` loops — one to draw a vertical line and one to draw a horizontal line. Both run in `setup()` because the pattern is static.

```cpp
#include "TinyFilmFestival.h"

void setup()
{
    ledBegin();
    
    // Vertical line through center (x=5)
    for (int y = 0; y < 8; y++)
    {
        ledWrite(5, y, HIGH);
    }
    
    // Horizontal line through center (y=3)
    for (int x = 0; x < 12; x++)
    {
        ledWrite(x, 3, HIGH);
    }
}

void loop()
{
    // Static pattern — nothing needed here
}
```

---

## Method 2 — Animation Mode

Animation Mode lets you play pre-made frame-by-frame animations. You create animations visually in the [LED Matrix Editor](https://ledmatrix-editor.arduino.cc/) and export them as `.h` files that your Arduino code can play back.

> [Full Animation Mode Documentation](https://digitalfuturesocadu.github.io/TinyFilmFestival/docs/#animation-mode)

### Key Functions

| Function | Description |
|----------|-------------|
| `screen.begin()` | Initialize the screen (call once in `setup()`) |
| `screen.play(anim, mode)` | Play an animation with a playback mode |
| `screen.update()` | Advance the animation — **must be called every `loop()`** |
| `screen.pause()` | Pause the current animation |
| `screen.resume()` | Resume a paused animation |
| `screen.stop()` | Stop the animation and clear the display |
| `screen.setSpeed(multiplier)` | Change playback speed (1.0 = normal, 2.0 = double, 0.5 = half) |

### Play Modes

| Mode | Description |
|------|-------------|
| `LOOP` | Play continuously, restart when finished |
| `ONCE` | Play once and stop on the last frame |
| `BOOMERANG` | Play forward then backward (ping-pong) |

### Creating an Animation

1. Open the [LED Matrix Editor](https://ledmatrix-editor.arduino.cc/)
2. Draw frames by painting pixels — use **Brush** (`B`) and **Eraser** (`E`)
3. Add new frames with `Ctrl + N`, duplicate with `Ctrl + D`
4. Preview your animation with `Spacebar`
5. **Export** the animation as a `.h` file (click the download button or `Ctrl + E`)
6. Copy the `.h` file into the same folder as your `.ino` sketch

> **Important:** Do not include spaces in the file name when exporting.

### Understanding the .h File

When you export an animation, the `.h` file contains a variable (usually named `animation`). You reference this name in your code to load the animation:

```cpp
#include "TinyFilmFestival.h"
#include "myAnimation.h"       // Your exported file

TinyScreen screen;
Animation myAnim = animation;  // 'animation' is the variable name inside the .h file

// --- Playback Settings (adjust these!) ---
PlayMode playMode = LOOP;           // LOOP, ONCE, or BOOMERANG
float playSpeed = 1.0;         // 1.0 = normal, 2.0 = double, 0.5 = half

void setup()
{
    screen.begin();
    screen.play(myAnim, playMode);
    screen.setSpeed(playSpeed);
}

void loop()
{
    screen.update(); // Required every loop!
}
```

> **Tip:** Open your `.h` file in a text editor to check the variable name if you're unsure.

### Example: Play Animation in Boomerang Mode at Half Speed

```cpp
#include "TinyFilmFestival.h"
#include "myAnimation.h"

TinyScreen screen;
Animation myAnim = animation;

// --- Playback Settings (adjust these!) ---
PlayMode playMode = BOOMERANG;      // LOOP, ONCE, or BOOMERANG
float playSpeed = 0.5;         // 0.5 = half speed for slow, smooth playback

void setup()
{
    screen.begin();
    screen.play(myAnim, playMode);
    screen.setSpeed(playSpeed);
}

void loop()
{
    screen.update();
}
```

### Example: Play a Partial Clip (Frames 2–6)

```cpp
#include "TinyFilmFestival.h"
#include "myAnimation.h"

TinyScreen screen;
Animation myAnim = animation;

// --- Playback Settings (adjust these!) ---
PlayMode playMode = LOOP;           // LOOP, ONCE, or BOOMERANG
float playSpeed = 1.0;         // 1.0 = normal speed
int startFrame = 2;            // First frame to play (1-indexed)
int endFrame = 6;              // Last frame to play

void setup()
{
    screen.begin();
    screen.play(myAnim, playMode, startFrame, endFrame);
    screen.setSpeed(playSpeed);
}

void loop()
{
    screen.update();
}
```

---

## Method 3 — Canvas Mode

Canvas Mode lets you draw shapes and text directly on the matrix using code. It uses a double-buffered drawing system — you draw between `beginDraw()` and `endDraw()`, and the result appears on the matrix all at once.

> [Full Canvas Mode Documentation](https://digitalfuturesocadu.github.io/TinyFilmFestival/docs/#canvas-mode)

### Key Concepts

- The matrix is **monochrome** — LEDs are either `ON` or `OFF` (no color or brightness)
- Drawing commands go between `screen.beginDraw()` and `screen.endDraw()`
- Use `screen.background(OFF)` to clear the screen each frame
- Use `screen.stroke(ON)` to set outlines to lit

### Drawing Functions

| Function | Description |
|----------|-------------|
| `screen.beginDraw()` | Start a drawing operation |
| `screen.endDraw()` | Finish and display the drawing |
| `screen.background(OFF)` | Clear the screen (set all LEDs off) |
| `screen.stroke(ON)` | Set outlines/points to lit |
| `screen.fill(ON)` | Set shape fills to lit |
| `screen.noFill()` | Outline only (no fill) |
| `screen.point(x, y)` | Draw a single point |
| `screen.line(x1, y1, x2, y2)` | Draw a line between two points |
| `screen.rect(x, y, w, h)` | Draw a rectangle |
| `screen.circle(cx, cy, diameter)` | Draw a circle |
| `screen.text("string", x, y)` | Draw text at a position |
| `screen.scrollText("string", x, y)` | Draw scrolling text |
| `screen.setScrollSpeed(ms)` | Set scrolling speed in ms per pixel |

### Example: Static Rectangle

```cpp
#include "TinyFilmFestival.h"

TinyScreen screen;

void setup()
{
    screen.begin();
}

void loop()
{
    screen.beginDraw();
    screen.background(OFF);
    screen.stroke(ON);
    screen.noFill();
    screen.rect(2, 1, 8, 6); // Rectangle outline
    screen.endDraw();
}
```

### Example: Dot Moving Across the Screen

This example uses `millis()` to move the dot at a steady pace without blocking.

```cpp
#include "TinyFilmFestival.h"

TinyScreen screen;
int x = 0;
unsigned long previousMillis = 0;
const long moveInterval = 100; // Move every 100ms

void setup()
{
    screen.begin();
}

void loop()
{
    unsigned long currentMillis = millis();

    if (currentMillis - previousMillis >= moveInterval)
    {
        previousMillis = currentMillis;
        x = (x + 1) % 12; // Move right, wrap around
    }

    screen.beginDraw();
    screen.background(OFF);
    screen.stroke(ON);
    screen.point(x, 4); // Dot at center row
    screen.endDraw();
}
```

The line `x = (x + 1) % 12` does two things:
- `x + 1` moves the dot one pixel to the right
- `% 12` is the **modulo** operator — it gives the remainder after dividing by 12. So when `x` reaches 12, `12 % 12 = 0`, and the dot wraps back to the left edge

| x before | x + 1 | % 12 | Result |
|----------|-------|------|--------|
| 0 | 1 | 1 | Moves right |
| 5 | 6 | 6 | Moves right |
| 10 | 11 | 11 | Moves right |
| 11 | 12 | **0** | **Wraps to start** |

This is a common pattern for looping a value within a range.

### Example: Bouncing Dot Using oscillateInt()

`oscillateInt()` smoothly cycles an integer between a min and max value over a period of time. It's a simple way to create animation without manually tracking position or direction.

```cpp
#include "TinyFilmFestival.h"

TinyScreen screen;

void setup()
{
    screen.begin();
}

void loop()
{
    // oscillateInt(min, max, periodMs) — cycles automatically based on time
    int x = oscillateInt(0, 11, 2000); // Left to right over 2 seconds
    int y = oscillateInt(0, 7, 1500);  // Top to bottom over 1.5 seconds
    
    screen.beginDraw();
    screen.background(OFF);
    screen.stroke(ON);
    screen.point(x, y);
    screen.endDraw();
}
```

### Example: Draw Multiple Shapes

```cpp
#include "TinyFilmFestival.h"

TinyScreen screen;

void setup()
{
    screen.begin();
}

void loop()
{
    screen.beginDraw();
    screen.background(OFF);
    screen.stroke(ON);
    screen.noFill();
    
    screen.point(0, 0);           // Single dot top-left
    screen.line(0, 7, 11, 0);     // Diagonal line
    screen.rect(8, 4, 4, 4);      // Small rectangle bottom-right
    screen.circle(3, 4, 4);       // Circle on the left
    
    screen.endDraw();
}
```

### Example: Animated Circle with Expanding Diameter

```cpp
#include "TinyFilmFestival.h"

TinyScreen screen;

void setup()
{
    screen.begin();
}

void loop()
{
    int diameter = oscillateInt(1, 7, 2000); // Grows and shrinks
    
    screen.beginDraw();
    screen.background(OFF);
    screen.stroke(ON);
    screen.noFill();
    screen.circle(5, 3, diameter);
    screen.endDraw();
}
```

### Example: Scrolling Text

```cpp
#include "TinyFilmFestival.h"

TinyScreen screen;

void setup()
{
    screen.begin();
}

void loop()
{
    screen.beginDraw();
    screen.background(OFF);
    screen.stroke(ON);
    screen.setScrollSpeed(80);             // 80ms per pixel
    screen.scrollText("Hello World!", 0, 1);
    screen.endDraw();
}
```

### Example: Two Dots Moving in Opposite Directions

Using the `offset` parameter of `oscillateInt()`, you can create elements that move out of phase with each other.

```cpp
#include "TinyFilmFestival.h"

TinyScreen screen;

void setup()
{
    screen.begin();
}

void loop()
{
    int y1 = oscillateInt(0, 7, 1500);        // Normal phase
    int y2 = oscillateInt(0, 7, 1500, 0.5);   // Opposite phase (offset = 0.5)
    
    screen.beginDraw();
    screen.background(OFF);
    screen.stroke(ON);
    screen.point(4, y1);
    screen.point(7, y2);
    screen.endDraw();
}
```

---

## Putting It All Together

You now have the tools to complete today's workshop. Each part of the workshop uses one of the 3 methods above:

| Workshop Part | Method | Key Starting Example |
|--------------|--------|---------------------|
| Part 1 — Install / Test | Simple LED | File → Examples → TinyFilmFestival → 01_Basics → SimpleLED |
| Part 2 — Create / Play Animation | Animation Mode | File → Examples → TinyFilmFestival → 01_Basics → FirstAnimation |
| Part 3 — Draw on the Canvas | Canvas Mode | File → Examples → TinyFilmFestival → 01_Basics → FirstCanvas |

Use the examples and API references above to go beyond the built-in examples and create something of your own. Consult the [full documentation](https://digitalfuturesocadu.github.io/TinyFilmFestival/docs/#home) for additional functions and features.

## Deliverable

Workshop 1 submission due by next class. Submit via the Canvas link as a Discussion Post.


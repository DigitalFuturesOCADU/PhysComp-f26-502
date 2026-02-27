# Class 07 | February 27 | Workshop 3

[← Back to Tiny Screens](../TinyScreens.md)

## Overview

This is the last workshop before the [Tiny Screens](../TinyScreens.md) exhibition (Class 08). In [Class 05](class05-Feb06.md) you learned output tools, and in [Class 06](class06-Feb13.md) you connected sensors to those visuals. Today you’ll combine these skills: starting from **either** a sensor technique or a matrix technique and building toward a complete input→output interaction.

The code sections today assume you're comfortable with the [read → translate → draw](class06-CodePatterns.md#the-loop-pattern) pattern and potentially with [writing functions](class06-CodePatterns.md#writing-functions) from Class 06. If not start with the [Quick Refresher](#quick-refresher) below.

### What We'll Cover

1. [Quick Refresher](#quick-refresher) — core Class 06 concepts in one table
2. [Advanced Sensor Techniques](#advanced-sensor-techniques) — calibration, smoothing, velocity & direction (input only — no screen output)
3. [Advanced Matrix Examples](#advanced-matrix-examples) — more complex Canvas and Animation patterns (output only — no sensor input)
4. [Enclosure & Fabrication](#enclosure--fabrication) — design, diffusers, wiring, sensor placement
5. [Workshop](#workshop) — what to do today
6. [Submission](#submission) — what to turn in


## Resources

- [TinyFilmFestival Documentation](https://digitalfuturesocadu.github.io/TinyFilmFestival/docs/#home)
- [EasyUltrasonic Library](https://github.com/SpulberGeorge/EasyUltrasonic)
- [LED Matrix Editor](https://ledmatrix-editor.arduino.cc/)
- [Class 06 Code Patterns](class06-CodePatterns.md)
- [Class 06 Workshop 2 Notes](class06-Feb13.md)
- [Class 05 Workshop 1 Notes](class05-Feb06.md)
- [Delay vs Millis](../../01-AltController/DelayVsMillis.md)

---

## Quick Refresher

It's been a few weeks. Here's a one-table summary of the core patterns from Class 06. If you need more detail, follow the links.

| Concept | One-Sentence Reminder | Reference |
|---|---|---|
| **Read → Translate → Draw** | Every `loop()` reads the sensor, converts the value, and updates the screen | [The Loop Pattern](class06-CodePatterns.md#the-loop-pattern) |
| **`map()` + `constrain()`** | `map()` converts one range to another; `constrain()` clips values that go out of bounds | [map() Reference](https://docs.arduino.cc/language-reference/en/functions/math/map/) |
| **`if/else` Thresholds** | Split a sensor range into zones — one threshold = two zones, two = three zones | [Name Your Thresholds](class06-CodePatterns.md#name-your-thresholds) |
| **Variable Naming** | Name values by meaning (`gripStrength`, `distanceToFace`), not by type (`val`, `reading`) | [Input Language](class06-CodePatterns.md#input-language) |
| **Functions** | Keep `loop()` readable — put logic in named functions like `readDistance()`, `drawCircle(x)` | [Writing Functions](class06-CodePatterns.md#writing-functions) |
| **Non-blocking Timing** | Use `millis()` instead of `delay()` so sensor reads and screen updates never freeze | [Delay vs Millis](../../01-AltController/DelayVsMillis.md) |

### Sensor Wiring Quick Reference

| Sensor | Pins | Library |
|---|---|---|
| **HC-SR04 Distance** | VCC→5V, GND→GND, TRIG→A0, ECHO→A1 | `EasyUltrasonic` — `ultrasonic.attach()`, `ultrasonic.getDistanceCM()` |
| **Custom FSR Pressure** | Layer 1→5V, Layer 2→A0 + 220Ω→GND | None — `analogRead(A0)` returns 0–1023 |

---

## Advanced Sensor Techniques

These files each demonstrate a technique that makes sensor input **more reliable or more expressive**. Each is a single complete sketch that reads and processes sensor data and prints results to Serial Monitor. 

They follow the same [read → translate](class06-CodePatterns.md#the-loop-pattern) pattern from Class 06, so focus on **what’s new** in each one.

### Pressure Sensor

- [Auto-Calibration at Startup](class07-pressure_calibration.md)
    - **What:** Reads the sensor during `setup()` to find its resting baseline, then sets thresholds relative to that value.
    - **Why:** Every custom FSR is different, and readings shift between sessions. Calibration means your thresholds work every time without manually editing constants.
    - **Key concept:** `baseline + offset = threshold`

- [Rolling Average Smoothing](class07-pressure_smoothing.md)
    - **What:** Keeps the last N readings in an array and averages them, producing a stable value instead of jittery raw input.
    - **Why:** Raw `analogRead()` values jump around frame-to-frame. Smoothing removes the noise so your visual output feels intentional, not twitchy.
    - **Key concept:** Circular buffer with running sum

- [Calibration + Smoothing Combined](class07-pressure_calibration_smoothing.md)
    - **What:** Combines both techniques — calibrate at startup for accurate thresholds, then smooth every reading for stable output.
    - **Why:** Calibration handles per-build variation; smoothing handles per-frame noise. Together they give you the most reliable pressure input.
    - **Key concept:** The two techniques solve different problems and stack cleanly

### Distance Sensor

- [Velocity and Direction](class07-distance_velocity.md)
    - **What:** Calculates how fast an object is moving and whether it's approaching or retreating, using two consecutive distance readings and the time between them.
    - **Why:** Raw distance tells you *where* something is. Velocity tells you *how it's moving* — which enables gesture-like interactions (fast approach vs. slow retreat).
    - **Key concept:** `velocity = change in distance / change in time`

---

## Advanced Matrix Examples

These examples use **no sensor input** — they demonstrate more complex Canvas and Animation techniques using the TinyFilmFestival library. Each one is self-contained and works as-is. As you read them, think about **which parameters a sensor could replace** — every `oscillateInt()` value or hard-coded constant is a potential sensor input.

### Canvas Examples

- [Animated Checkerboard](class07-canvas_checkerboard.md)
    - **What:** A checkerboard pattern that shifts over time using `oscillateInt()` and nested `for` loops.
    - **Why:** Introduces grid-based drawing with modulo (`%`) — a single offset value controls the entire pattern, making it easy to swap in a sensor later.
    - **Key concepts:** Nested `for` loops, modulo alternation, `oscillateInt()`

- [Bouncing Shapes with Phase Offsets](class07-canvas_bouncing_shapes.md)
    - **What:** Three shapes — circle, rectangle, dot — each animated independently with different periods and phase offsets.
    - **Why:** Shows how multiple `oscillateInt()` calls with phase offsets create layered, organic motion without any sensor input. Each call is a potential `map()` replacement.
    - **Key concepts:** Multiple `oscillateInt()` calls, phase offsets (0.0–1.0), canvas drawing order

- [Dot Trail with History Array](class07-canvas_dot_trail.md)
    - **What:** A moving dot leaves a trail using a circular buffer (array) to remember its last N positions.
    - **Why:** The circular buffer is the same pattern used in [rolling average smoothing](class07-pressure_smoothing.md) — learning it here gives you two uses for one concept. A trail also makes motion history visible.
    - **Key concepts:** Circular buffer, `millis()` timing for snapshots, `for` loop to draw history

- [Smooth Motion with Ease](class07-canvas_ease_motion.md)
    - **What:** A circle eases between target positions with smooth acceleration/deceleration, pausing briefly at each stop. Size also eases — shrinking while moving, growing when stopped.
    - **Why:** Unlike `oscillateInt()` which cycles forever, `Ease` moves to a target once and stops — making it ideal for state-driven or sensor-triggered motion.
    - **Key concepts:** `Ease` class, `.to(target, duration)`, `.done()` for sequencing, `.intValue()` for pixel coordinates

### Animation Examples

The examples below use ready-made `.h` files from the [TinyFilmFestival example animations](https://github.com/DigitalFuturesOCADU/TinyFilmFestival/tree/main/exampleAnimations). Download the ones you need and place them in the same folder as your `.ino` sketch. Each example includes a note on how to swap in a different animation — just change the `#include` and variable name. See the [Animation Mode guide](https://digitalfuturesocadu.github.io/TinyFilmFestival/docs/#animation-mode) for full documentation.

- [Layered Animations](class07-animation_layers.md)
    - **What:** Two animations play simultaneously on separate layers, each at its own speed.
    - **Why:** Layers let you composite multiple animations — one could be a background loop while the other responds to sensor input. Each layer has independent speed and play-mode control.
    - **Key concepts:** `addLayer()`, `playOnLayer()`, `setSpeedOnLayer()`

- [Sprite Animation with setPosition](class07-animation_position.md)
    - **What:** An animation plays while `oscillateInt()` slides it across the matrix like a sprite.
    - **Why:** Separates *what* an animation looks like from *where* it appears. Position is the most natural parameter to control with a distance sensor.
    - **Key concepts:** `setPosition(x, y)`, animation playback + dynamic positioning

- [Sequential Animation Playback](class07-animation_sequence.md)
    - **What:** Two animations play in sequence — A finishes, then B starts, then back to A — using `ONCE` mode and `isComplete()`.
    - **Why:** Introduces state tracking and completion detection. The automatic trigger (`isComplete()`) can easily be swapped for a sensor threshold to make transitions interactive.
    - **Key concepts:** `ONCE` play mode, `isComplete()`, state variable for tracking

---

## Enclosure & Fabrication

Your project must be enclosed for exhibition. No bare LEDs, no exposed wiring. This section covers practical strategies.

### Enclosure Types

| Type | Good For | Considerations |
|---|---|---|
| **Free-standing** | Tabletop objects, sculptures | Needs a stable base; hide Arduino inside or behind |
| **Wearable** | Body-mounted interactions | Keep lightweight; secure wiring for movement |
| **Wall/surface-mounted** | Installations, directional sensing | Consider viewing angle and sensor orientation |

### Magnifiers & Diffusers

The LED matrix must be covered — no bare LEDs visible. Two options that can be combined:

- **Magnifier** — a small lens (like those from a dollar store magnifying glass) placed over the matrix enlarges the pixel image. Creates a clear, bright display. Position matters: move it closer or farther from the matrix to adjust focus and size.
- **Diffuser** — translucent material (vellum, parchment paper, thin fabric, frosted acrylic) placed over the matrix softens the dots into glowing regions. Creates a softer, more ambient display. Thicker or more layers = more diffusion.

Combine both: diffuser directly on the matrix, magnifier on top for a large, soft display.

### Sensor Placement

- **Distance sensor (HC-SR04):** The sensor needs a clear line of sight — no obstructions between the sensor face and the interaction zone. Point it in the direction you want to sense. Consider mounting angle: facing up (hand hovers above), facing out (hand approaches from front), facing down (object placed below).
- **Pressure sensor (FSR):** Must be sandwiched between something rigid (the enclosure surface) and something the user touches. Consider where on the object people will naturally press, grip, or squeeze.

### Wire Management

- Use **short wires** where possible — less slack = fewer tangles
- **Secure connections** with tape or hot glue (on the insulation, not the metal)
- Route wires **inside** the enclosure walls, not dangling through openings
- Leave enough slack at the Arduino end to remove and reconnect if needed
- **Test after enclosing** — a wire that worked loose ruins a demo

### Power

- **Laptop USB** — most reliable for exhibition; ensures Serial Monitor access for debugging
- **Battery pack** — frees the object from a computer; use a USB battery bank with the Arduino's USB-C port. Test runtime before exhibition day.

---

## Workshop

### Part 1 — Combine Input + Output (Groups of 2–4)

Form a group of 2–4 people. Choose **one** starting point:

| Starting Point | What's Missing |
|---|---|
| An [Advanced Sensor Technique](#advanced-sensor-techniques) example | Screen output |
| An [Advanced Matrix Example](#advanced-matrix-examples) | Sensor input |

**How to think through it step by step:**

The [From Static to Dynamic](class06-CodePatterns.md#from-static-to-dynamic) guide in Class 06 walks through exactly this process:

1. **Identify the parameter** — What value in the sketch could change? In sensor examples, it’s the output visual. In matrix examples, it’s the hard-coded `oscillateInt()` values. See [Step 2: Identify the Parameter](class06-CodePatterns.md#step-2-identify-the-parameter).
2. **Replace with a named variable** — Give it a descriptive name (`gripStrength`, `handSpeed`). See [Step 3: Replace with a Named Variable](class06-CodePatterns.md#step-3-replace-with-a-named-variable).
3. **Drive it with the missing half** — If you started with a sensor example, write a draw function. If you started with a matrix example, add sensor reads and `map()`. See [Step 4: Drive the Variable with Sensor Data](class06-CodePatterns.md#step-4-drive-the-variable-with-sensor-data).
4. **Use the [read → translate → draw](class06-CodePatterns.md#the-loop-pattern) pattern** to structure your `loop()`.

Refer to the [Sensor Wiring Quick Reference](#sensor-wiring-quick-reference) above for pin connections.

### Part 2 — Project Sketch + Plan (Individual)

Create two drawings/diagrams that describe what your device will look like and how it will work at next week’s exhibition:

1. **Device sketch** — A drawing or rendering of what your physical object/wearable will look like. Include the enclosure, sensor placement, and screen position. Review [Enclosure & Fabrication](#enclosure--fabrication) for guidance.
2. **Interaction diagram** — A simple diagram that shows:
    - The **preposition** you plan to use (from [Interaction as Preposition](../TinyScreens.md#interaction-as-preposition))
    - How the **input** (sensor) will affect what appears **on screen** (the cause → effect relationship)

---

## Submission

Create a **single post** on Canvas covering both parts. This is an **individual** submission.

### Part 1 — Group Exercise

- **Image** showing the input/output your group created (photo or screenshot of the interaction working)
- **One sentence** describing what the sensor controls and how it appears on screen

### Part 2 — Project Plan

- **Image 1:** Sketch of your planned device (drawing, rendering, or annotated photo of materials)
- **Image 2:** Interaction diagram showing your preposition + how input affects output

### Checklist Before You Submit

- [ ] Part 1 image clearly shows sensor input connected to screen output
- [ ] Part 1 sentence describes the interaction (e.g., “Grip pressure controls circle size on the matrix”)
- [ ] Part 2 sketch shows the physical form of my device with sensor and screen placement
- [ ] Part 2 diagram names my preposition and shows the input → output relationship

# Sensor + Servo — What You Can Build

[← Back to Light & Lightness](LightAndLightness.md)

---

This page maps everything you can do with light sensors and servos in this project. Each section links to a focused guide with wiring, code, and step-by-step instructions. Start wherever makes sense — you do not need to work through these in order, but everything assumes you have completed the core [Light Sensor to Servo](classes/class10-LDR-to-Servo.md) walkthrough first.

---

## Foundations

These two reference documents cover the building blocks for every pattern on this page.

### Movement Functions

Two reusable functions for controlling servo motors. Both are non-blocking — they run alongside sensor reads and other logic in your main loop without freezing anything.

| Function | What It Does | Character |
|---|---|---|
| `oscillate()` | Sweeps back and forth continuously using a sine wave | Rhythmic, breathing, pendulum-like |
| `moveServoA()` | Moves to a target angle over a set duration, then signals completion | Deliberate, sequential, choreographed |

`oscillate()` is stateless — one function works for any number of servos. `moveServoA()` is stateful — each servo needs its own copy of the function.

**Full documentation:** [Servo Movement Reference](servo-movement-reference.md)

### Sensor Reading

A photoresistor (LDR) reads ambient light as a voltage. The Arduino converts this to a number from 0 (dark) to 1023 (bright). The actual range depends on your environment — use the Serial Monitor to observe your values before mapping or setting thresholds.

Beyond raw readings, you can smooth noisy data with a rolling average and calibrate at startup to adapt to whatever room the piece is running in.

**Full documentation:** [Light Sensor Guide](classes/LightSensorGuide.md)

---

## Connecting Sensor to Movement

The design work lives in the relationship between what the sensor reads and what the servo does. There are three fundamental patterns, and every behavior you build is some variation of one of them.

### Continuous Mapping

`map()` translates the sensor's range directly into a movement parameter — angle, speed, or range of motion. Any change in light produces a proportional change in the output. The connection is immediate and fluid.

- Light level controls servo position
- Light level controls oscillation speed
- Light level controls range of motion

**Walkthrough:** [Light Sensor to Servo — Part 4](classes/class10-LDR-to-Servo.md#part-4--connect-sensor-to-servo)
**Reference:** [Light Sensor Guide — Continuous Mapping](classes/LightSensorGuide.md#analogread--map--continuous-mapping) · [Servo Reference — oscillate() + map()](servo-movement-reference.md#oscillate--map--sensor-controls-speed-or-range)

### Threshold Switching

Instead of a proportional response, a threshold divides the sensor range into states. Cross the boundary and the servo's entire behavior changes — different speed, different range, different character. Extend to multiple thresholds for more zones.

- Dark → slow, wide sweep / Bright → fast, narrow tremor
- Dark → move to one position / Bright → move to another
- Three zones → three distinct movement personalities

**Walkthrough:** [Light Sensor to Servo — Part 5 (oscillation)](classes/class10-LDR-to-Servo.md#part-5--thresholds-and-oscillation) · [Part 6 (timed moves)](classes/class10-LDR-to-Servo.md#part-6--thresholds-and-timed-moves)
**Reference:** [Light Sensor Guide — Threshold Switching](classes/LightSensorGuide.md#analogread--if--threshold-switching) · [Servo Reference — oscillate() + if()](servo-movement-reference.md#oscillate--if--threshold-switches-between-behaviours)

### Two-Sensor Comparison

With two photoresistors, the question changes from "how bright is it?" to "which side is brighter?" The servo responds to the relationship between two readings — a directional, relational input.

- Servo follows the brighter side
- Object orients toward a light source
- Neutral zone when both sides are equal (deadband)

**Reference:** [Light Sensor Guide — Two Sensors](classes/LightSensorGuide.md#two-sensors--which-side-is-brighter) · [Servo Reference — Which Sensor Is Brighter](servo-movement-reference.md#oscillate--which-of-two-sensors-is-brighter)

---

## Sequencing and Choreography

`moveServoA()` returns the exact target angle when a move is complete. This lets you chain moves into sequences using `switch/case` or arrays.

- **switch/case** — Hardcode a fixed sequence of moves (angles and durations). Each case advances to the next on completion.
- **Array sequences** — Store targets and durations in arrays. Index into them with a step counter. Swap between different arrays based on a sensor threshold.

These are the tools for building choreographed, time-based movement rather than continuous oscillation.

**Walkthrough:** [Servo Basics — Part 4 (timed moves)](classes/class09-ServoBasics.md) · [Servo Basics — Part 5 (switch/case)](classes/class09-ServoBasics.md)
**Reference:** [Servo Reference — Sequencing Moves](servo-movement-reference.md#sequencing-moves-with-switchcase) · [Servo Reference — Array Sequences](servo-movement-reference.md#moveservoa--arrays--sequences-selected-by-condition)

---

## Sensor Techniques

These techniques apply to any photoresistor reading, regardless of what you do with the value afterward.

### Smoothing

A rolling average reduces noise from the sensor by averaging recent readings. One function call wraps the raw `analogRead()` and returns a stable value. Useful when small fluctuations cause the servo to jitter.

**Reference:** [Light Sensor Guide — Smoothing](classes/LightSensorGuide.md#3-smoothing--stabilizing-noisy-readings)

### Calibration

Read the sensor at startup to establish a baseline for the current environment. Thresholds and mappings are then relative to that baseline rather than hardcoded to a specific room. The piece adapts automatically when moved to a new location.

**Reference:** [Light Sensor Guide — Calibration](classes/LightSensorGuide.md#4-calibration--measuring-ambient-light-at-startup)

---

## Working with Multiples

A single sensor and a single servo is the starting point. Adding a second of either opens up new possibilities for how the object reads its environment and how it moves.

### Multiple Servos

Two servos can operate simultaneously with independent behaviors — different oscillation speeds, different timed move sequences, one oscillating while the other does a choreographed sequence. Each servo needs its own `Servo` object, its own pin, and its own copy of `moveServoA()` (named `moveServoB()` for the second).

**Step-by-step guide:** [Multiple Servos](classes/MultipleServos.md)
**Function reference:** [Servo Reference — moveServoB()](servo-movement-reference.md#3-moveservob--second-servo-timed-move)

### Multiple Light Sensors

Two photoresistors let you read light from two locations simultaneously. Compare the readings to detect directionality — which side is brighter, how large the difference is, whether both sensors agree or disagree.

**Step-by-step guide:** [Multiple Light Sensors](classes/MultipleLDRs.md)

### Two Sensors + Two Servos — The Full Setup

With two LDRs and two servos, the design space expands combinatorially. Each sensor can drive either servo (or both), and each servo can run a different movement function. Paired mapping, crossed mapping, comparison-driven behaviors, mixed modes, and asymmetric configurations are all possible.

**Patterns and combinations:** [Two LDRs + Two Servos](classes/TwoLDRsTwoServos.md)

---

## Physical Adjustments

Sensors and servos are connected to the Arduino with short jumper wires. For many designs, you will want the servo or sensor mounted away from the breadboard — on a wall, inside an enclosure, or at the end of an arm. This requires extending wires using solid core wire from the fabrication lab.

**Practical guide:** [Physical Adjustments — Extending Wires](classes/PhysicalAdjustments.md)

---

## Design Considerations

### Movement as Material

The servo produces rotation. What you build from that — through code, mechanism, and material — defines the character of your piece. Think in terms of qualities: speed, range, rhythm, character, responsiveness. Describe the movement you want in words before writing the code to produce it.

### Light and Shadow as Feedback

Your object does not just respond to light — it casts shadows and blocks light as it moves. This can create a feedback loop: the servo moves, the shadow shifts, the sensor reads the change, and the servo responds again. This is a design feature, not a bug. Consider whether you want to amplify or dampen it.

### Sensor Placement

Where the sensor sits determines what the object is sensitive to. External and exposed reads the room. Inside an enclosure reads the object's own shadow. At the base reads the surface. On opposite sides of the form enables directional sensing. The placement decision is inseparable from what the object does.

### Materials

Paper, vellum, fabric, and thin materials respond dramatically to small servo movements. Heavy materials resist and dampen. Translucent materials interact with light differently than opaque ones. Your material choices determine the quality of light your piece produces.

See: [Light & Lightness — Movement as Material](LightAndLightness.md#movement-as-material)

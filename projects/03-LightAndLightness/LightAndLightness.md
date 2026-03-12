# Light & Lightness

<!-- TODO: Add main project image (light/shadow kinetic object) -->

## Project Overview

This project focuses on creating responsive kinetic objects that manipulate light and shadow through movement. You will use servo motors to control mechanisms built primarily from lightweight materials — paper, vellum, fabric — and read input from a light sensor to affect and change its behavior over time. You should consider its behavior for the entire 30 minute span of the open exhibition. How can it adapt and change over that time? How can you accentuate and expand the physical and visual properties of your chosen material by controlling it with the servo?

The input to your design will come from 1 or more light sensors. These sensors are highly versatile and can be used to read a range of interactions based on their location and orientation. Consider how to embed them into your overall design in a way that focuses the interaction between people, space, light, and the object itself.

Rather than working with a complicated library, you will build the servo response using simple functions that can be layered and expanded to create complex movements. Full documentation in [servo-movement-reference.md](servo-movement-reference.md).

| | |
|---|---|
| **Keywords** | Servos, Mechanisms, Light Sensors, Kinetic Objects |
| **Format** | Individual |
| **Movement Reference** | [Servo Movement Functions](servo-movement-reference.md) |
| **Resources** | [Light Sensor Guide](classes/LightSensorGuide.md), [Delay vs Millis](../01-AltController/DelayVsMillis.md) |

---

## Sections

- [Class Pages](#class-pages)
- [From Screen to Space](#from-screen-to-space--changes-and-similarities-from-project-2)
- [Movement as Material](#movement-as-material)
- [Hardware](#hardware)
- [Coding Concepts](#coding-concepts)
- [Iterative Prototyping](#iterative-prototyping)
- [Lecture Slides](#lecture-slides)

---

## Class Pages

| Date | Topic |
|------|-------|
| March 13 | [Class 09 - Workshop 1](classes/class09-Mar20.md) |
| March 20 | [Class 10 - Workshop 2](classes/class10-Mar27.md) |
| March 27 | [Class 11 - Workshop 3](classes/class11-Apr03.md) |
| April 3 | [Class 12 - Project Exhibition](classes/class12-Apr03.md) |
| April 8 | Documentation Due |

---

## Lecture Slides

| Class | Slides |
|-------|--------|
| Class 09 | *Link to slides* |
| Class 10 | *Link to slides* |
| Class 11 | *Link to slides* |
| Class 12 | *Link to slides* |

---

## From Screen to Space : Changes and Similarities from Project 2

### Physical Output vs. Screen Output

In Project 2, output was visual — pixels on a 12x8 LED matrix. In this project, output is physical — a servo motor rotating a mechanism in three-dimensional space. This introduces new concerns: torque (can the servo move your mechanism?), range of motion (a servo sweeps 0°–180°), speed (physical objects have momentum), and sound (servos and movement make noise). It also provides new opportunities for the relationship between the object, viewer, and light source.

### Analog Input: Light vs. Pressure

Both the pressure sensor (Project 2) and the light sensor (Project 3) are read with `analogRead()` and return values in the 0–1023 range. Pressure was direct and gestural — someone actively squeezes or presses. Light is environmental and ambient — the sensor reads the room, a shadow, a passing hand. The interaction can be subtle and indirect or require active engagement. You must also consider the feedback loop of how your object manipulates the light hitting the sensor.

### Mapping and Thresholds

The same `map()` function and threshold logic from Project 2 apply directly. What changes is what you are mapping to: sensor → servo angle, sensor → sweep speed or cycle duration, sensor → range of motion, threshold → change movement pattern. We will once again use these core interaction concepts, but look at how they differ based on the type of input and output.

### Timing

Similar to Project 2, controlling the timing of reading input and the speed of output is critical. `millis()` remains an important tool for developing an interactive system that can be highly responsive, but update data at different rates.

See: [Delay vs Millis](../01-AltController/DelayVsMillis.md)

---

## Movement as Material

Just as Project 2 used a *preposition* to define the relationship between person and object, this project asks you to think about **movement as a design material** — something with qualities you choose deliberately.

A servo rotates back and forth. That is the raw material. What you build with it — through code, mechanism, and material — defines the character of your piece.

### Qualities of Movement

Think about the movement your object makes and describe it in words before writing code:

<!-- TODO: Consider adding specific examples linking quality words to oscillate() / moveServo() parameter choices -->

| Quality | Description | Code Lever |
|---|---|---|
| **Speed** | fast / slow / accelerating / decelerating | Cycle duration, move duration |
| **Range** | wide sweep / narrow wobble / full rotation | Min/max angles |
| **Rhythm** | continuous / intermittent / syncopated | Timing, state machine transitions |
| **Character** | smooth / jerky / hesitant / aggressive | Sine vs. linear motion, pause durations |
| **Responsiveness** | immediate / gradual / delayed / accumulating | How sensor input maps to movement parameters |

### Light and Shadow as Feedback

Your object does not just move — it casts shadows, blocks light, creates patterns on walls and surfaces. The movement *produces* visual output in physical space. Consider:

- Does the object's shadow tell a different story than the object itself?
- Does the shadow grow, shrink, rotate, fragment?
- Can you position a light source to amplify or direct the shadow?
- Does the object's own movement change the light reaching its sensor? (this creates a feedback loop)

<!-- TODO: Add visual examples/reference images of kinetic shadow art -->

### Lightness of Materials

The word "lightness" has a double meaning in this project:

1. **Physical lightness** — Paper, vellum, fabric, and thin materials respond dramatically to small servo movements. A slight rotation can make a sheet of paper billow, a vellum panel cast a moving shadow, or a fabric strip ripple. Heavy materials resist and dampen movement.

2. **Optical lightness** — These same materials interact with light in specific ways. Vellum is translucent. Paper reflects and shadows sharply. Fabric diffuses. Your material choices determine the *quality of light* your piece produces.

<!-- TODO: Add material examples with photos — paper, vellum, fabric, wire, thread -->
<!-- TODO: Consider adding references to kinetic art artists (Calder, Tinguely, Theo Jansen, etc.) -->

### Mechanisms

Mechanisms are another key tool in shaping the character of motion. Servos output rotational movement, but this can be amplified or altered through mechanisms. Well-designed mechanisms provide opportunities that aren't possible with code alone, such as converting rotational movement into linear.

<!-- TODO: Add diagrams or references for basic mechanism types (crank-slider, four-bar linkage, cam, etc.) -->

---

## Hardware

### Input

- **Light Sensor (Photoresistor)** — Analog sensor read with `analogRead()`. Wired in a voltage divider with a 10kΩ resistor. Returns 0 (dark) to 1023 (bright). We will also be investigating how the possibilities of where the sensors are placed can be expanded by soldering longer wires to it. See: [Light Sensor Guide](classes/LightSensorGuide.md) for full details

### Output

- **Servo Motors** — Minimum one, additional servos optional. We will be focused on the [Micro Servos](https://www.dfrobot.com/product-1579.html) included in your kit. These servos are small and are not particularly powerful, which is why the project focuses on manipulating lightweight materials. Work within the constraints of what the motors can do. See: [Servo Movement Reference](servo-movement-reference.md)
- ***Optional* LEDs or other light sources** — The primary output should come from the servos, but you do have the option of also using external LEDs or the LED Matrix

---

## Coding Concepts

### Reading a Light Sensor

The photoresistor is an analog sensor — no library required. Use `analogRead()` to get values from 0 (dark) to 1023 (bright):

```cpp
int lightValue = analogRead(A0);
```

The actual range depends on your environment. Use the Serial Monitor to observe your values before mapping or setting thresholds. We will also examine how concepts of smoothing, calibration, and relative data apply to light data. See: [Light Sensor Guide](classes/LightSensorGuide.md)

### Servo Movement Functions

Rather than working with a complicated library, you will build the servo response using simple functions that can be layered and expanded to create complex movements. Full documentation in [servo-movement-reference.md](servo-movement-reference.md).

| Function | Motion Type | State | Multi-Servo | Use Case |
|---|---|---|---|---|
| `oscillate()` | Continuous sine-wave sweep | Stateless — calculates from `millis()` | One function for all servos | Ongoing, rhythmic motion |
| `moveServo()` | Timed point-to-point | Stateful — uses `static` variables | One copy per servo | Sequenced, choreographed moves |

- **`oscillate(minAngle, maxAngle, cycleMs)`** — Returns an angle that sweeps smoothly between two values. The motion naturally slows at extremes, like a pendulum. Set-and-forget: call it every loop and it runs forever.

- **`moveServo(angle, durationMs)`** — Moves from the current position to a target angle over a set duration. Returns the target angle when complete, allowing you to chain moves using `switch`/`case`.

### Mapping Sensor Data to Movement

Use `map()` to translate light sensor values into movement parameters — the same pattern from Project 2, with different targets:

```cpp
// Light controls the speed of oscillation
int cycleSpeed = map(lightValue, 200, 800, 500, 3000);
int angle = oscillate(30, 150, cycleSpeed);

// Light controls the range of motion
int maxAngle = map(lightValue, 200, 800, 90, 180);
int angle = oscillate(30, maxAngle, 2000);
```

See: [Code Patterns — map()](../02-TinyScreens/classes/class06-CodePatterns.md)

### Thresholds for Behavior Changes

Break your light sensor range into zones that trigger different movement behaviors:

```cpp
if (lightValue < 300) {
  // Dark — slow, wide sweep
  angle = oscillate(20, 160, 4000);
} else if (lightValue < 600) {
  // Medium — moderate movement
  angle = oscillate(60, 120, 2000);
} else {
  // Bright — fast, narrow twitch
  angle = oscillate(80, 100, 400);
}
```

See: [Code Patterns — Thresholds](../02-TinyScreens/classes/class06-CodePatterns.md)

### Timing with millis()

Both `oscillate()` and `moveServo()` are already non-blocking — they use `millis()` internally. This means your `loop()` can read sensors, update servos, and manage state without anything waiting or blocking.

If you need additional timed events (pauses between movements, periodic sensor reads, state transitions on a timer), use the same `millis()`-based timing pattern from previous projects.

See: [Delay vs Millis](../01-AltController/DelayVsMillis.md)

### State Machines for Movement Sequences

Use `switch`/`case` with `moveServo()` completion to choreograph movement sequences. The light sensor can drive state transitions — changing which sequence plays, how fast it runs, or when it switches:

```cpp
switch (movementState) {
  case 0:
    angleA = moveServoA(180, 1500);
    if (angleA == 180) movementState = 1;
    break;
  case 1:
    angleA = moveServoA(10, 2000);
    if (angleA == 10) movementState = 2;
    break;
  case 2:
    angleA = moveServoA(90, 500);
    if (angleA == 90) movementState = 0;
    break;
}
```

<!-- TODO: Add an example showing light sensor input driving state transitions -->

---

## Iterative Prototyping

This project uses a deliberate iterative process to evolve your ideas over the 4 weeks. Starting with class 09, you will create your first concept drawings for the project alongside a working interaction between the sensor and the servos. From there you will be asked to iterate and update that design each week, meaning that what is shown at the exhibition is *at least* version 4. This method of development is standard practice within Physical Computing as a means to create refined results.

### Workshop 1 — Foundation (March 13)

**Goal:** Get the parts working independently.

- Read the light sensor and observe values in Serial Monitor
- Connect a servo and control it with `oscillate()` and/or `moveServo()`
- Begin connecting sensor input to servo output (basic mapping)
- Sketch your concept: what moves? what material? what does light do?

<!-- TODO: Link to class09-Mar20.md when workshop content is written -->

### Workshop 2 — Integration (March 20)

**Goal:** Connect sensor to movement with intention. Build the mechanism.

- Refine the relationship between light input and movement output
- Build your physical mechanism — attach lightweight materials to the servo
- Develop at least two distinct movement behaviors (e.g., light vs. dark responses)
- Test how your materials respond to the servo's motion
- Begin considering light source and shadow placement

<!-- TODO: Link to class10-Mar27.md when workshop content is written -->

### Workshop 3 — Refinement (March 27)

**Goal:** Refine movement, materials, and enclosure.

- Fine-tune movement patterns and sensor responsiveness
- Refine material construction and enclosure
- Test light and shadow behavior
- Prepare for exhibition

### Exhibition — Polish (April 3)

**Goal:** Finished, exhibitable piece.

- Final enclosure: all electronics enclosed, wires managed
- Light and shadow working as intended
- Movement is expressive and responsive to the light sensor
- Object works reliably for extended periods

### Documentation (April 8)

- Full documentation on your website, covering concept, process, and final result

<!-- TODO: Add documentation requirements and format expectations -->

---

## Design Constraints

- Primary materials for the moving elements must be **lightweight**: paper, vellum, fabric, thread, wire, or similar
- Must **manipulate light and shadow** as part of its expression — movement should create visible changes in light/shadow
- Must use a **light sensor** as input — the sensor's reading must affect movement behavior over time
- Create **unconventional kinetic movements** — do not leave the servo as a simple back-and-forth sweep with no mechanical transformation
- Consider the **mechanical strength and limitations** of your servo motors — do not overload them with heavy or rigid materials
- All electronics and wiring **enclosed** and managed
- Write your own code — use the movement functions as a starting point, not a final product

## Do Not

- Use heavy or rigid materials (wood, acrylic, metal) as your primary moving elements
- Leave the servo doing a simple untransformed sweep — use a mechanism, linkage, or code to create something more expressive
- Ignore the light sensor in the final piece — it must actively affect movement
- Glue your enclosure shut — use connections that allow access for maintenance
- Use `delay()` in your code — use `millis()`-based timing
- Create an object that is based around stuffed animals
- Create an object that is based around a weapon

<!-- TODO: Review and expand this list — are there common student mistakes to preempt? -->

---

## Design Considerations

### Mechanism

How does the rotational motion of a servo become the movement of your piece? A servo turns — but your object could wave, nod, flutter, pulse, or twist. Consider linkages, cams, cranks, pulleys, levers, or direct drive. The mechanism *is* the design.

<!-- TODO: Add diagrams or references for basic mechanism types (crank-slider, four-bar linkage, cam, etc.) -->

### Responsiveness

How does the object's behavior change with light? Is the change gradual (mapped) or sudden (threshold)? Predictable or surprising? Does it become more active in light or in darkness? How does this choice relate to the concept of your piece?

### Materiality

How do paper, vellum, or fabric amplify the servo's small rotational movement? A 30° sweep might barely be visible with a rigid arm — but it could make a sheet of paper billow across a wide arc. Choose materials that extend and transform the motion, not just carry it.

### Light & Shadow

Is light the subject of the work, or the medium? Are you sculpting shadow? Filtering light? Creating moving patterns? Consider where your light source is, how your materials interact with it, and whether the sensor reads the room's light or the object's own shadow.

### Craft & Finish

The work should feel intentional and complete. Electronics are enclosed. Wires are managed. Materials are cut and attached with care. The servo and mechanism work reliably. The overall presentation communicates that every choice was deliberate.

<!-- TODO: Add references to enclosure techniques, wire management strategies -->
<!-- TODO: Consider adding exhibition setup requirements (table, power, lighting) -->

---

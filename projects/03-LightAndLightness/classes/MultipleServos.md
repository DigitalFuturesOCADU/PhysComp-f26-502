# Multiple Servos — Step by Step

[← Back to Sensor + Servo Guide](../SensorServoGuide.md) · [← Back to Light & Lightness](../LightAndLightness.md)

---

This guide adds a second servo to your circuit and shows how to run both independently at the same time. It assumes you have completed [Servo Basics](class09-ServoBasics.md) and are comfortable with `oscillate()` and `moveServoA()`.

Two stages:

1. **Wire the second servo** — add it to the breadboard and confirm it moves independently.
2. **Run both servos with independent behaviors** — oscillation, timed moves, mixed modes.

---

## Part 1 — Wire the Second Servo

Your first servo is already connected to pin 9. The second servo connects to a different signal pin but shares the same 5V and GND rails.

| Servo B Wire | Purpose | Arduino Connection |
|------------|---------|-------------------|
| **Brown** | GND | GND power rail (shared) |
| **Red** | Power | 5V power rail (shared) |
| **Orange** | Signal | Pin 10 |

Use jumper cables to connect each wire, same as the first servo.

### Wiring Diagram

![Two servos on a breadboard](../assets/2Servos__breadboard_bb.png)

> **Check before moving on:** Both servos share the same 5V and GND rails. Servo A signal goes to pin 9, Servo B signal goes to pin 10.

---

## Part 2 — Set Up Both Servos in Code

Start from your working single-servo sketch. You need a second `Servo` object, a second pin variable, and a second angle variable.

### Step 1 — Add the Second Servo Object and Variables

Below your existing global variables, add the new ones for Servo B:

```cpp
Servo myServoB;
int servoPinB = 10;
int angleB    = 90;
```

**What is happening here:**
- `myServoB` is a separate servo object. The Arduino talks to each servo through its own object.
- `servoPinB` stores the signal pin for the second servo.
- `angleB` tracks the second servo's position independently from `angle` (which tracks Servo A).

### Step 2 — Attach Both Servos in setup()

Add the attach call for Servo B:

```cpp
void setup() {
  Serial.begin(9600);
  myServo.attach(servoPin);
  myServoB.attach(servoPinB);
}
```

**What is happening here:**
- Each servo needs its own `attach()` call. Without it, `write()` calls for that servo have no effect.

### Step 3 — Write Both Angles in loop()

Make sure your `loop()` writes to both servos:

```cpp
myServo.write(angle);
myServoB.write(angleB);
```

**What is happening here:**
- Each servo receives its own angle value. They can be the same number or completely different — the servos are independent.

### Upload and Test

Upload. Both servos should move to 90° and hold. Try changing `angleB` to a different value and re-uploading to confirm the second servo responds independently.

> **Check your sketch:** You should have two `Servo` objects, two pin variables, and two angle variables. Both servos are attached in `setup()` and written in `loop()`.

---

## Part 3 — Independent Oscillations

Both servos can use the same `oscillate()` function simultaneously because it is stateless — it calculates everything from `millis()` with no memory between calls.

### Step 4 — Add Separate Parameters for Each Servo

Above `setup()`, create a set of sweep variables for each servo:

```cpp
// Servo A — slow, wide sweep
int sweepMinA    = 30;
int sweepMaxA    = 150;
int sweepPeriodA = 3000;

// Servo B — fast, narrow tremor
int sweepMinB    = 80;
int sweepMaxB    = 100;
int sweepPeriodB = 400;
```

### Step 5 — Call oscillate() Twice in loop()

```cpp
void loop() {
  angle  = oscillate(sweepMinA, sweepMaxA, sweepPeriodA);
  angleB = oscillate(sweepMinB, sweepMaxB, sweepPeriodB);

  myServo.write(angle);
  myServoB.write(angleB);
}
```

**What is happening here:**
- Each call to `oscillate()` gets its own set of parameters. The function does not know or care which servo will receive the result — it just returns a number.
- Servo A sweeps slowly across a wide arc. Servo B trembles rapidly in a narrow range. Both run simultaneously in the same `loop()`.

Upload and observe. Two servos moving with completely different characters from the same function.

> **Check your sketch:** Both servos oscillate independently. Changing one servo's parameters does not affect the other.

---

## Part 4 — Independent Timed Moves

`moveServoA()` uses `static` variables to remember where a move started, where it is going, and when it began. Those static variables belong to the function — there is only one copy of them. If two servos try to share the same function, the second call overwrites the first servo's stored state.

The solution: give each servo its own copy of the function. The code inside is identical — only the function name and the `myServo.read()` call change.

### Step 6 — Add moveServoB()

You already have `moveServoA()` in your sketch (from [Servo Basics](class09-ServoBasics.md)). Below it, add a second copy for Servo B:

```cpp
int moveServoB(int angle, unsigned long durationMs) {

  static float startAngle        = 0.0;
  static int   targetAngle       = -1;
  static unsigned long startTime = 0;

  if (angle != targetAngle) {
    startAngle  = myServoB.read();  // reads Servo B's position
    targetAngle = angle;
    startTime   = millis();
  }

  unsigned long elapsed = millis() - startTime;
  float progress = (float)elapsed / (float)durationMs;

  if (progress >= 1.0) {
    targetAngle = -1;
    return angle;
  }

  float current = startAngle + (angle - startAngle) * progress;
  return (int)current;
}
```

**What is happening here:**
- This is an exact copy of `moveServoA()` with two changes: the function name is `moveServoB` and the internal `read()` call uses `myServoB` instead of `myServo`.
- The `static` variables inside `moveServoB()` are entirely separate from those inside `moveServoA()`. Each function has its own private whiteboard. Neither can see or overwrite the other's state.

### Step 7 — Set Start Positions in setup()

Both servos need a known starting position so that `read()` returns a reliable value on the first move:

```cpp
void setup() {
  Serial.begin(9600);
  myServo.attach(servoPin);
  myServoB.attach(servoPinB);
  myServo.write(homeAngle);
  myServoB.write(homeAngleB);
}
```

Add `homeAngleB` to your global variables if you want the two servos to start at different positions.

### Step 8 — Run Independent Sequences

Each servo can run its own `switch/case` sequence with its own step counter. In your global variables:

```cpp
int sequenceStepA = 0;
int sequenceStepB = 0;
```

In `loop()`, the two sequences run side by side:

```cpp
void loop() {

  // Servo A sequence
  switch (sequenceStepA) {
    case 0:
      angle = moveServoA(10, 1500);
      if (angle == 10) { sequenceStepA = 1; }
      break;
    case 1:
      angle = moveServoA(170, 2000);
      if (angle == 170) { sequenceStepA = 0; }
      break;
  }

  // Servo B sequence
  switch (sequenceStepB) {
    case 0:
      angleB = moveServoB(160, 800);
      if (angleB == 160) { sequenceStepB = 1; }
      break;
    case 1:
      angleB = moveServoB(20, 3000);
      if (angleB == 20) { sequenceStepB = 0; }
      break;
  }

  myServo.write(angle);
  myServoB.write(angleB);
}
```

**What is happening here:**
- The two `switch/case` blocks are completely independent. Each advances its own step counter based on its own move function's return value.
- Servo A moves at its own pace with its own targets. Servo B does the same. They run in parallel — both are updated every loop, but each is on its own timeline.
- The sequences do not need the same number of steps or the same durations. One can be fast and simple, the other slow and complex.

---

## Part 5 — Mixed Modes

The two servos do not need to use the same movement approach. One can oscillate continuously while the other runs a timed sequence, or one can respond to a sensor threshold while the other follows a fixed pattern.

### Step 9 — One Oscillating, One Sequencing

```cpp
void loop() {
  // Servo A: continuous oscillation
  angle = oscillate(sweepMinA, sweepMaxA, sweepPeriodA);

  // Servo B: timed move sequence
  switch (sequenceStepB) {
    case 0:
      angleB = moveServoB(30, 2000);
      if (angleB == 30) { sequenceStepB = 1; }
      break;
    case 1:
      angleB = moveServoB(150, 1000);
      if (angleB == 150) { sequenceStepB = 0; }
      break;
  }

  myServo.write(angle);
  myServoB.write(angleB);
}
```

**What is happening here:**
- Servo A uses `oscillate()` — set-and-forget, runs forever.
- Servo B uses `moveServoB()` with `switch/case` — a choreographed sequence that loops.
- Both run in the same `loop()` without interfering. The movement functions handle their own timing internally.

This is the core principle: each servo gets its own angle variable, its own movement function call, and its own parameters. You compose behaviors by deciding what drives each servo independently.

---

## Part 6 — Explore

You now have two servos running independently from two movement functions. The design work is in the combination.

### Things to try

- **Same function, different parameters** — both oscillating but with deliberately different speeds and ranges. What does the interplay feel like?
- **Opposing motion** — one moves left while the other moves right. Swap the min/max values or offset the ranges.
- **Call and response** — one servo finishes a move, then the other begins. Use the completion check from `moveServoA()`/`moveServoB()` to trigger the other.
- **Sensor-driven** — if you have a light sensor connected, use a threshold to change one servo's behavior while the other stays constant. See [Two LDRs + Two Servos](TwoLDRsTwoServos.md) for expanded patterns.

### Reference

- [Servo Movement Reference](../servo-movement-reference.md) — full function documentation for `oscillate()`, `moveServoA()`, and `moveServoB()`
- [Sensor + Servo Guide](../SensorServoGuide.md) — overview of all patterns and techniques

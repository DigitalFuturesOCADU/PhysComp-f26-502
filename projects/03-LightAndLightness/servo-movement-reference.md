# Arduino Servo Movement Functions

Two reusable functions for controlling servo motors: one for continuous oscillation, one for timed point-to-point moves. Both are **non-blocking**, so they work alongside sensor reads and other logic in your main loop.

---

## 1. `oscillate()` — Sine Wave Oscillation

Smoothly sweeps a servo back and forth between two angles using a sine wave. The motion naturally slows at the extremes, like a pendulum.

### The Function

```cpp
/*
 * oscillate()
 *
 * Returns an angle that smoothly sweeps between minAngle and maxAngle
 * over the given cycle duration, using a sine wave.
 *
 * This function is STATELESS — it calculates everything fresh from
 * millis() each call, so one function can safely be shared by
 * multiple servos.
 *
 *   minAngle — lowest angle in the sweep (0–180)
 *   maxAngle — highest angle in the sweep (0–180)
 *   cycleMs  — time for one full back-and-forth oscillation (ms)
 */
int oscillate(int minAngle, int maxAngle, int cycleMs) {

  // Current time as a fraction of the cycle (0.0 → 1.0, then repeats)
  float progress = (float)(millis() % cycleMs) / cycleMs;

  // Convert progress to a sine value (-1.0 to 1.0)
  float sineValue = sin(progress * TWO_PI);

  // Remap: shift sine from (-1 to 1) → (0 to 1)
  float normalized = (sineValue + 1.0) / 2.0;

  // Scale to our angle range
  float angle = minAngle + normalized * (maxAngle - minAngle);

  return (int)angle;
}
```

### How It Works

1. `millis()` gives the current time in milliseconds
2. We divide by the cycle duration to get how far through the cycle we are
3. Multiply by `TWO_PI` to convert to radians (one full sine cycle)
4. `sin()` gives a value from -1.0 to 1.0
5. We remap that range into our min/max angle range

Because the function calculates everything from the current time with no memory of previous calls, it is **stateless**. This means a single function can safely be shared by multiple servos — just call it with different settings for each one.

### Full Example — Two Servos Oscillating Independently

```cpp
#include <Servo.h>

Servo servoA;
Servo servoB;

const int SERVO_A_PIN = 9;
const int SERVO_B_PIN = 10;


void setup() {
  servoA.attach(SERVO_A_PIN);
  servoB.attach(SERVO_B_PIN);
}


void loop() {

  // Both servos can use the same function with different settings
  int angleA = oscillate(30, 150, 2000);   // wide sweep, 2 second cycle
  int angleB = oscillate(70, 110, 800);    // narrow wobble, fast cycle

  servoA.write(angleA);
  servoB.write(angleB);

  delay(20);
}
```

---

## 2. `moveServo()` — Timed Move to Target Angle

Moves a servo from its current position to a target angle over a specified number of milliseconds. Returns the current angle as an int, which you pass to `servo.write()`. When the returned angle matches your target, the move is done.

### The Function

```cpp
/*
 * moveServoA()
 *
 * Call this every loop() with your desired target angle and duration.
 *   - First call with a new target: reads the servo's current position
 *   - Subsequent calls: returns the interpolated position
 *   - When complete: returns the exact target angle
 *
 * Uses myServo.read() to grab the starting position automatically.
 *
 *   angle      — the target angle (0–180)
 *   durationMs — how long the move should take in milliseconds
 */
int moveServoA(int angle, unsigned long durationMs) {

  // These are static — they persist between calls.
  // This is servoA's private whiteboard.
  static float startAngle    = 0.0;
  static int   targetAngle   = -1;
  static unsigned long startTime = 0;

  // New target? Start a fresh move.
  if (angle != targetAngle) {
    startAngle  = servoA.read();
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

### Why Each Servo Needs Its Own Function

This function uses **`static` variables** to remember the state of a move between calls. Unlike the oscillation function, it is **stateful** — it needs to track where a move started, where it's going, and when it began.

`static` variables persist between function calls (they are created once and never reset), but there is only **one copy** per function. If two servos shared the same function, they'd overwrite each other's state.

Think of it like a whiteboard in a meeting room. A `static` variable is a whiteboard with a "DO NOT ERASE" sign — your notes survive between meetings. But if two different teams share the same whiteboard, they'll overwrite each other's plans. Separate functions = separate whiteboards.

### Full Example — Two Servos Moving Independently

```cpp
#include <Servo.h>

Servo servoA;
Servo servoB;

int sequenceStep = 0;


void setup() {
  servoA.attach(9);
  servoB.attach(10);
  servoA.write(90);
  servoB.write(90);
}


void loop() {

  int angleA, angleB;

  // --- Servo A: runs a 3-step sequence ---
  // Each case is one move. When it finishes, advance to the next step.
  switch (sequenceStep) {
    case 0:
      angleA = moveServoA(180, 1500);
      if (angleA == 180) sequenceStep = 1;
      break;
    case 1:
      angleA = moveServoA(10, 2000);
      if (angleA == 10) sequenceStep = 2;
      break;
    case 2:
      angleA = moveServoA(90, 500);
      if (angleA == 90) sequenceStep = 0;
      break;
  }

  // --- Servo B: independently moves to 45° over 3 seconds ---
  angleB = moveServoB(45, 3000);

  servoA.write(angleA);
  servoB.write(angleB);
}


// =============================================
//  moveServoA — controls servoA only
// =============================================
int moveServoA(int angle, unsigned long durationMs) {

  static float startAngle    = 0.0;
  static int   targetAngle   = -1;
  static unsigned long startTime = 0;

  if (angle != targetAngle) {
    startAngle  = servoA.read();
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


// =============================================
//  moveServoB — controls servoB only
// =============================================
// Identical logic to moveServoA. Each copy has its own static
// variables (its own whiteboard), so the two servos don't
// interfere with each other.
int moveServoB(int angle, unsigned long durationMs) {

  static float startAngle    = 0.0;
  static int   targetAngle   = -1;
  static unsigned long startTime = 0;

  if (angle != targetAngle) {
    startAngle  = servoB.read();
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

---

## Key Concepts Comparison

| | `oscillate()` | `moveServo()` |
|---|---|---|
| **Motion type** | Continuous back-and-forth | Point-to-point |
| **State** | Stateless — calculates from `millis()` each call | Stateful — uses `static` variables to track progress |
| **Multi-servo** | One function works for all servos | Each servo needs its own copy of the function |
| **Completion** | Never "finishes" — loops forever | Returns exact target angle when done |
| **Sequencing** | Set-and-forget | Use `switch`/`case` to chain moves |

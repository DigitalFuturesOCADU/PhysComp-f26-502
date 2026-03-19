# Light Sensor to Servo — Step by Step

[← Back to Workshop 1](class09-Mar13.md) · [← Back to Light & Lightness](../LightAndLightness.md)

---

Five stages today:

1. **Wire and read the light sensor** — get the photoresistor on a breadboard and confirm it sends values to the Serial Monitor.
2. **Wire and test the servo** — add a servo motor to the circuit and confirm it moves to a position you set in code.
3. **Connect the two** — use `map()` to translate the sensor reading into a servo angle so light controls position.
4. **Thresholds and oscillation** — replace the continuous map with a threshold check that triggers different `oscillate()` behaviors in each state.
5. **Thresholds and timed moves** — swap `oscillate()` for `moveServoA()` so the servo travels to different fixed positions depending on the light level.

Each stage is self-contained. Test and confirm that things work before moving on to the next.

---

## Part 1 — Wire the Light Sensor

A photoresistor (also called an LDR — Light Dependent Resistor) changes its resistance depending on how much light hits it. The Arduino cannot read resistance directly — it reads voltage. So we wire the photoresistor in a **voltage divider** with a fixed resistor, which converts the changing resistance into a changing voltage.

### About the Breadboard

- The **two long rows** along the top and bottom edges are **power rails**. All holes in one rail are connected to each other. Use these for **5V** and **GND**.
- The **short rows** in the middle connect across each row of five holes, but not across the center gap.
- Push component legs and wires into the holes to create connections.

### Wiring

| Component | Connection |
|-----------|------------|
| **Photoresistor — Leg 1** | 5V power rail |
| **Photoresistor — Leg 2** | Same row as the wire to **A0** and one leg of the 10kΩ resistor |
| **10kΩ Resistor — other leg** | GND power rail |
| **Arduino 5V** | 5V power rail |
| **Arduino GND** | GND power rail |

### Wiring Diagram

![LDR breadboard wiring diagram](../assets/1LDR_breadboard_bb.png)

![LDR breadboard wiring detail](../assets/1LDR_breadboard_detail.png)

> **Check before moving on:** The photoresistor has one leg at 5V and one leg connecting to both the A0 wire and one leg of the 10kΩ resistor. The other leg of the resistor goes to GND.

---

## Part 2 — Read the Sensor

Open a **new Arduino sketch** (File → New Sketch). We will build the code one step at a time.

### Step 1 — Create Variables

Above `setup()`, add:

```cpp
int lightPin   = A0;
int lightValue = 0;
```

**What is happening here:**
- `lightPin` stores the pin number. Using a variable name means we only change one line if the wire moves later.
- `lightValue` will hold the sensor reading each loop. We set it to `0` as a placeholder.

### Step 2 — Open Serial, Read, and Print

Inside `setup()`, open the Serial connection. Inside `loop()`, read the sensor and print it:

```cpp
void setup() {
  Serial.begin(9600);
}

void loop() {
  lightValue = analogRead(lightPin);

  Serial.print("Light: ");
  Serial.println(lightValue);
}
```

**What is happening here:**
- `Serial.begin(9600)` opens the connection between the Arduino and your computer at 9600 baud. This lets us see what the sensor is doing in the Serial Monitor.
- `analogRead(lightPin)` reads the voltage on A0 and converts it to a number from **0 to 1023**. The result replaces whatever was in `lightValue`.
- `Serial.print("Light: ")` sends the label without a line break. `Serial.println(lightValue)` sends the number and then moves to a new line.

### Upload and Test

Upload the sketch. Open the **Serial Monitor** (Tools → Serial Monitor) and make sure the baud rate at the bottom right is set to **9600**.

Numbers should update continuously. Cover the sensor — they drop. Uncover it — they rise. Note the rough low and high values you see in your environment. You will need those numbers in Part 4.

> **Check your sketch:** You should have two global variables, a `setup()` with one Serial line, and a `loop()` with one read and two print lines. Numbers should change in the Serial Monitor when you cover and uncover the sensor.

---

## Part 3 — Add the Servo

Now add the servo motor to the circuit. A servo has three wires bundled into a single plug:

![Servo connector showing brown, red, and orange wires](../assets/servoPins.jpg)

| Servo Wire | Purpose | Arduino Connection |
|------------|---------|-------------------|
| **Brown** | GND | GND power rail |
| **Red** | Power | 5V power rail |
| **Orange** | Signal | Pin 9 |

The servo shares the same 5V and GND rails the photoresistor is already using. The servo plug does not fit directly into the breadboard — use pin-to-pin jumper cables to connect each wire:

![Using jumper cables to connect the servo](../assets/servoPinsWjumpers.jpg)

### Wiring Diagram

![Breadboard wiring with LDR and servo](../assets/1Servo_1LDR_breadboard_bb.png)

> **Check before moving on:** Servo brown wire to GND rail, red to 5V rail, orange to pin 9.

### Step 3 — Include the Servo Library and Add Variables

At the very top of your sketch, before your existing variables, add the library include. Below your existing variables, add the servo variables:

```cpp
#include <Servo.h>

// lightPin and lightValue stay here ...

Servo myServo;
int servoPin = 9;
int angle    = 90;
```

**What is happening here:**
- `#include <Servo.h>` loads Arduino's built-in Servo library. Without this line the Arduino does not know what a `Servo` is.
- `Servo myServo;` creates a servo object — a named thing we can send instructions to.
- `angle = 90` is the starting position. 90 is the midpoint of the servo's 0–180° range.

### Step 4 — Attach in setup(), Write in loop()

In `setup()`, attach the servo. In `loop()`, write the angle to the servo and update the print statements to show both values:

```cpp
void setup() {
  Serial.begin(9600);
  myServo.attach(servoPin);
}

void loop() {
  lightValue = analogRead(lightPin);

  Serial.print("Light: ");
  Serial.print(lightValue);
  Serial.print(" | Angle: ");
  Serial.println(angle);

  myServo.write(angle);
}
```

**What is happening here:**
- `myServo.attach(servoPin)` tells the Servo library which pin to send signals on. Without this, `write()` calls have no effect.
- `myServo.write(angle)` tells the servo to move to the position in `angle`. Right now that is always 90.
- We print the angle alongside the light reading so we can watch both values change once they are connected.

### Upload and Test

Upload. The servo should move to 90° and hold. Attach one of the servo horns (the small plastic arms that clip onto the shaft) so you can clearly see the position. Try changing `int angle = 90;` to `10`, then `170` — upload each time and watch the servo move. Then set it back to `90`.

> **Check your sketch:** You should have `#include <Servo.h>` at the top, five global variables, a `setup()` that opens Serial and attaches the servo, and a `loop()` that reads the sensor, prints both values, and writes the angle. The servo holds at 90° and the Serial Monitor shows both values.

---

## Part 4 — Connect Sensor to Servo

The sensor and servo both work, but `angle` is still a fixed `90`. Now we make the light value control the servo position using `constrain()` and `map()`.

### Step 5 — Add Range Variables

Above `setup()`, add four variables. Replace the placeholder numbers with the actual low and high values you noted from the Serial Monitor:

```cpp
int lightMin = 200;   // sensor covered — replace with your observed low value
int lightMax = 800;   // sensor uncovered — replace with your observed high value
int angleMin = 0;
int angleMax = 180;
```

**What is happening here:**
- `lightMin` and `lightMax` define the range your sensor actually produces in your environment. Using your real observed values means the mapping will use the full angle range rather than a fraction of it.
- `angleMin` and `angleMax` set how far the servo moves in response. `0` to `180` is the full sweep. Try `60` to `120` for a narrower, more centered movement.

### Step 6 — Constrain and Map

In `loop()`, add two lines after the `analogRead` line and before the print statements:

```cpp
void loop() {
  lightValue = analogRead(lightPin);

  lightValue = constrain(lightValue, lightMin, lightMax);
  angle = map(lightValue, lightMin, lightMax, angleMin, angleMax);

  Serial.print("Light: ");
  Serial.print(lightValue);
  Serial.print(" | Angle: ");
  Serial.println(angle);

  myServo.write(angle);
}
```

**What is happening here:**
- `constrain(lightValue, lightMin, lightMax)` clamps the reading so it never goes below `lightMin` or above `lightMax`. Readings outside your observed range will not produce wild servo movements. The result is written back into `lightValue`.
- `map(lightValue, lightMin, lightMax, angleMin, angleMax)` scales the clamped reading proportionally into the angle range. A reading at the midpoint of the input range becomes the midpoint of the angle range. The result is stored in `angle`.
- The `myServo.write(angle)` line does not change — it already writes whatever is in `angle` to the servo.

### Upload and Test

Upload. The servo should respond to changes in light. Cover the sensor — the servo moves one way. Uncover it — it moves the other way.

- If the servo barely moves, narrow `lightMin` and `lightMax` to match the values you actually see in the Serial Monitor.
- If the motion direction is reversed, swap the last two arguments: `map(lightValue, lightMin, lightMax, angleMax, angleMin)`.

> **Check your sketch:** You should have nine global variables, a `setup()` that opens Serial and attaches the servo, and a `loop()` that reads, constrains, maps, prints, and writes. The servo should respond smoothly to light changes.

---

## Part 5 — Thresholds and Oscillation

Instead of mapping the sensor value continuously to an angle, we can check whether the light is above or below a threshold and trigger different servo behaviors. The `oscillate()` function from [Servo Basics](class09-ServoBasics.md) calculates everything fresh from `millis()` on every call — so you can call it with completely different parameters in each branch of an `if/else` and it switches immediately, with no cleanup needed.

### Step 7 — Replace the Range Variables with a Threshold

Above `setup()`, remove the four range variables and replace them with one:

```cpp
int lightThreshold = 512;
```

**What is happening here:**
- The threshold divides the sensor range into two states. Values below it trigger one behavior; values above trigger another.
- `512` is the midpoint of the 0–1023 range. Replace it with a value from your observed readings that sits at the crossover point between dark and bright in your space.

### Step 8 — Replace the Map with an if/else

In `loop()`, remove the `constrain` and `map` lines and replace them with an `if/else` that calls `oscillate()`:

```cpp
void loop() {
  lightValue = analogRead(lightPin);

  if (lightValue < lightThreshold) {
    angle = oscillate(30, 150, 2000);  // dark: wide, slow sweep
  } else {
    angle = oscillate(85, 95, 300);    // bright: tight, fast tremor
  }

  Serial.print("Light: ");
  Serial.print(lightValue);
  Serial.print(" | Angle: ");
  Serial.println(angle);

  myServo.write(angle);
}
```

**What is happening here:**
- In the dark branch, `oscillate()` gets wide limits and a slow period — a broad, relaxed sweep.
- In the bright branch, `oscillate()` gets narrow limits and a fast period — a tight, nervous tremor.
- Because `oscillate()` is stateless, switching between branches requires no reset. The motion adapts immediately to whoever called it and with what parameters.
- The `myServo.write(angle)` line is unchanged.

### Step 9 — Add the oscillate() Function

Below the closing `}` of `loop()`, paste the function:

```cpp
int oscillate(int minAngle, int maxAngle, int cycleMs) {

  float progress   = (float)(millis() % cycleMs) / cycleMs;
  float sineValue  = sin(progress * TWO_PI);
  float normalized = (sineValue + 1.0) / 2.0;
  float angle      = minAngle + normalized * (maxAngle - minAngle);

  return (int)angle;
}
```

**What is happening here:**
- `millis() % cycleMs` gives how far into the current cycle we are as a value from 0 to `cycleMs − 1`. Dividing by `cycleMs` converts it to a fraction from 0.0 to 1.0.
- Multiplying by `TWO_PI` and passing to `sin()` produces a smooth wave from −1.0 to 1.0.
- The last two lines shift and scale that wave into the specified angle range.
- No state is stored between calls — which is what makes it safe to call with different parameters in different branches.

### Upload and Test

Upload. The servo should sweep slowly and widely in darkness, then switch to a tight rapid tremor in bright light. Cover and uncover the sensor and watch the transition.

Adjust the parameters directly inside the two `oscillate()` calls to change the character of each state. For a three-zone response, add a second threshold and a third `else if` branch with its own `oscillate()` call.

> **Check your sketch:** You should have `lightPin`, `lightValue`, `myServo`, `servoPin`, `angle`, and `lightThreshold` as globals. Your `loop()` reads the sensor, branches on the threshold, calls `oscillate()` with different parameters in each branch, prints, and writes. The `oscillate()` function is defined below `loop()`.

---

## Part 6 — Thresholds and Timed Moves

`oscillate()` runs forever and never arrives at any particular position. `moveServoA()`, from [Servo Basics](class09-ServoBasics.md), moves the servo to a specific angle over a set duration and signals when it has arrived. Using it inside a threshold check sends the servo to a different target depending on the light level.

### Step 10 — Replace the oscillate() Calls with moveServoA()

In `loop()`, replace the two `oscillate()` calls:

```cpp
void loop() {
  lightValue = analogRead(lightPin);

  if (lightValue < lightThreshold) {
    angle = moveServoA(20, 1200);   // dark: move to 20° over 1.2 seconds
  } else {
    angle = moveServoA(160, 1200); // bright: move to 160° over 1.2 seconds
  }

  Serial.print("Light: ");
  Serial.print(lightValue);
  Serial.print(" | Angle: ");
  Serial.println(angle);

  myServo.write(angle);
}
```

**What is happening here:**
- `moveServoA(20, 1200)` moves the servo to 20° over 1200 milliseconds. It returns the interpolated position during travel and returns exactly `20` when the move is complete.
- When the light crosses the threshold, the `if/else` switches to calling `moveServoA()` with the other target. `moveServoA()` uses a `static` variable to detect when the target has changed — when it does, the function immediately starts a fresh move from wherever the servo currently sits.
- The servo does not snap between targets. It travels smoothly from its current position to the new target each time the light condition changes.

### Step 11 — Replace oscillate() with moveServoA()

Remove the `oscillate()` function below `loop()` and replace it with `moveServoA()`:

```cpp
int moveServoA(int angle, unsigned long durationMs) {

  static float startAngle        = 0.0;
  static int   targetAngle       = -1;
  static unsigned long startTime = 0;

  if (angle != targetAngle) {
    startAngle  = myServo.read();
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
- The three `static` variables persist between calls. `startAngle` records where the move began, `targetAngle` remembers the destination, and `startTime` records when the move started.
- When a new target arrives (`angle != targetAngle`), the function reads the servo's current position with `myServo.read()`, saves it as the new start, and begins timing from now.
- `progress` runs from 0.0 to 1.0 across the duration. At 1.0 the move is done — `targetAngle` resets to −1 so the next fresh target starts clean.
- Because this function uses `static` variables, it has private memory. If you add a second servo, duplicate the function and rename the copy `moveServoB()` so the two servos track their moves independently.

### Step 12 — Set a Known Start Position

In `setup()`, after `myServo.attach(servoPin)`, write the servo to a known starting angle:

```cpp
void setup() {
  Serial.begin(9600);
  myServo.attach(servoPin);
  myServo.write(90);
}
```

**What is happening here:**
- `moveServoA()` calls `myServo.read()` when a new move begins to find the current position. `read()` returns the last value written. Without this line, the first move starts from an unpredictable position.

### Upload and Test

Upload. The servo should travel toward the dark-state target when you cover the sensor and back toward the bright-state target when you uncover it, moving smoothly each time the light crosses the threshold.

- Try a short duration like `400` for a quick snap, or `3000` for a slow drift.
- Try setting both targets close together for subtle, responsive movement.
- Add a second threshold and a third `else if` branch with its own `moveServoA()` call and target angle to create three states.

> **Check your sketch:** You should have the same six global variables as Part 5. Your `setup()` now writes a starting position. Your `loop()` branches on the threshold and calls `moveServoA()` in each branch. The `moveServoA()` function is defined below `loop()`.

---

## Part 7 — Explore

You now have three ways to connect light to servo motion: continuous mapping, threshold-triggered oscillation, and threshold-triggered timed moves. Each gives the system a different quality of response to light.

Attach something to the servo horn — a pipe cleaner, wire, tape, or folded paper. Anything that extends the movement makes it readable and gives the servo character. A bare servo rotating is hard to read.

### Things to try

- **Change the character of each state** — in the threshold and oscillation sketch, try one state as a barely-visible tremor and the other as a full sweeping arc. How does the transition between states feel?
- **Tighten the threshold** — bring `lightThreshold` very close to your observed ambient reading. The servo will react to subtle shifts in light rather than only major changes.
- **Add a second threshold** — split the range into three zones and give each its own behavior. Add `lightLow` and `lightHigh` variables and use `else if` to create three branches, each with its own `oscillate()` or `moveServoA()` call.
- **Mix the techniques** — oscillate in one state, hold still in another. Use a slow timed move as a resting state and a fast oscillation as a triggered response.
- **Reattach the horn at a different angle** — pull the horn off, rotate the shaft, and clip it back at a new position. The same code range now maps to a different physical arc without changing a line of code.


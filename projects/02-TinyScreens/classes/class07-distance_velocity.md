# Distance Sensor — Velocity and Direction

[← Back to Class 07](class07-Feb27.md)

In [Class 06](class06-Feb13.md), the distance sensor told you **where** something is — a single number in centimeters. That's useful for positioning, but it doesn't capture **how** something is moving. Is the hand approaching fast? Retreating slowly? Hovering still?

**Velocity** answers this. By comparing two consecutive distance readings and the time between them, you can calculate speed and direction — enabling gesture-like interactions that respond to *movement quality*, not just position.

---

## The Concept

Velocity is the rate of change of distance over time:

$$\text{velocity} = \frac{\text{current distance} - \text{previous distance}}{\text{elapsed time}}$$

The **sign** of the velocity tells you the direction:
- **Negative velocity** → object is getting closer (distance decreasing)
- **Positive velocity** → object is moving away (distance increasing)
- **Near zero** → object is stationary (within a dead zone)

The **magnitude** (`abs(velocity)`) tells you the speed — how fast, regardless of direction.

```
Distance over time:

100cm ┤
      │                              ╱── moving away (positive velocity)
 50cm ┤         ╲                   ╱
      │          ╲── approaching   •── still (near-zero velocity)
 10cm ┤           ╲  (negative)
      │            •
      └──────────────────────────────── time →
```

### The Dead Zone

Sensors are never perfectly still. Even with a stationary object, readings fluctuate by ±1–2 cm. Without a dead zone, the direction would flicker between "approaching" and "retreating" constantly. The `STILL_THRESHOLD` defines a minimum speed below which we consider the object stationary.

```
Velocity:  -3.2   -0.1   +0.3   +5.1   -0.2   -4.8
            ↓       ↓      ↓      ↓       ↓      ↓
Status:   TOWARD  STILL  STILL  AWAY   STILL  TOWARD
                    ↑      ↑             ↑
              below STILL_THRESHOLD — ignored
```

---

## Complete Sketch

This sketch calculates velocity and direction from the distance sensor. It prints all values to Serial Monitor — **no screen output is included**. In the [workshop](#putting-it-all-together), your group will add visual output.

```cpp
// ============================================================
// Distance Sensor — Velocity and Direction
// ============================================================
// What this does:
//   Compares consecutive distance readings to calculate
//   how fast an object is moving and in which direction.
//   Prints velocity, speed, and direction to Serial Monitor
//   for testing.
//
// Why this matters:
//   Raw distance tells you WHERE something is.
//   Velocity tells you HOW it's moving — which enables
//   gesture-like interactions: fast approach = urgency,
//   slow retreat = calm, still = idle.
//
// Key concept:
//   velocity = (currentDistance - previousDistance) / elapsedTime
// ============================================================

#include <EasyUltrasonic.h>

EasyUltrasonic ultrasonic;

// --- Pin configuration ---
int TRIG_PIN = A0;
int ECHO_PIN = A1;

// --- Sensor range ---
int MIN_DISTANCE = 2;     // cm — closest reliable reading
int MAX_DISTANCE = 100;   // cm — farthest we care about

// --- Velocity settings ---
// STILL_THRESHOLD defines the dead zone — velocities below this
// magnitude are treated as "not moving." This prevents flicker
// from sensor noise when an object is stationary.
// Units: cm per second
float STILL_THRESHOLD = 1.5;

// --- Timing ---
int READ_INTERVAL = 50;         // ms — slightly slower for stable velocity
unsigned long lastRead = 0;

// --- Tracking values ---
float currentDistance = 0;       // this frame's reading
float previousDistance = 0;      // last frame's reading
unsigned long currentTime = 0;  // when this reading was taken
unsigned long previousTime = 0; // when the last reading was taken

// --- Calculated values ---
float velocity = 0;              // cm per second (signed)
float speed = 0;                 // absolute speed (always positive)
int direction = 0;               // -1 = approaching, 0 = still, 1 = retreating

// ── Read the distance sensor ──
float readDistance()
{
    return ultrasonic.getDistanceCM();
}

// ── Calculate velocity from two readings ──
// Returns velocity in cm/second. Negative = approaching.
float calculateVelocity(float current, float previous, unsigned long elapsedMs)
{
    if (elapsedMs == 0) return 0;   // avoid divide-by-zero

    // Convert elapsed time to seconds for cm/s units
    float elapsedSeconds = elapsedMs / 1000.0;

    return (current - previous) / elapsedSeconds;
}

// ── Determine direction from velocity ──
// Returns: -1 = approaching, 0 = still, 1 = retreating
int getDirection(float vel)
{
    if (abs(vel) < STILL_THRESHOLD)
    {
        return 0;   // within dead zone — treat as stationary
    }
    else if (vel < 0)
    {
        return -1;  // distance decreasing — object approaching
    }
    else
    {
        return 1;   // distance increasing — object retreating
    }
}

void setup()
{
    Serial.begin(9600);
    ultrasonic.attach(TRIG_PIN, ECHO_PIN, MIN_DISTANCE, MAX_DISTANCE);

    // Take an initial reading so previousDistance has a value
    previousDistance = readDistance();
    previousTime = millis();
}

void loop()
{
    if (millis() - lastRead >= READ_INTERVAL)
    {
        lastRead = millis();

        // --- READ ---
        currentDistance = readDistance();
        currentTime = millis();

        // --- CALCULATE VELOCITY ---
        unsigned long elapsed = currentTime - previousTime;
        velocity = calculateVelocity(currentDistance, previousDistance, elapsed);

        // --- DERIVE SPEED AND DIRECTION ---
        speed = abs(velocity);              // magnitude only
        direction = getDirection(velocity); // -1, 0, or 1

        // --- OUTPUT — print to Serial Monitor ---
        Serial.print("Dist: ");
        Serial.print(currentDistance);
        Serial.print(" cm | Vel: ");
        Serial.print(velocity);
        Serial.print(" cm/s | Dir: ");
        if (direction == -1) Serial.print("TOWARD");
        else if (direction == 0) Serial.print("STILL");
        else Serial.print("AWAY");
        Serial.print(" | Speed: ");
        Serial.println(speed);

        // --- SAVE FOR NEXT FRAME ---
        previousDistance = currentDistance;
        previousTime = currentTime;
    }
}
```

---

## What to Notice

- **`READ_INTERVAL` is 50ms here**, slightly slower than the 20ms used in Class 06. Velocity needs enough time between readings for a measurable change. Too fast (5ms) and the difference between readings is just noise. Too slow (500ms) and the response feels laggy. 50ms is a good starting point.
- **Division by zero protection** — `calculateVelocity()` checks for `elapsedMs == 0` before dividing. This is defensive coding that prevents crashes.
- **The dead zone (`STILL_THRESHOLD`) is critical.** Without it, a stationary hand would produce tiny positive and negative velocities from sensor noise, causing the direction to flicker constantly. The threshold filters this out.
- **`previousDistance` and `previousTime` are saved at the end of each read cycle**, so the next frame has a reference point. This is the simplest form of "memory" — remembering one frame back.
- **Speed and direction are separated.** `abs(velocity)` gives speed (always positive). The sign of `velocity` gives direction. This keeps the two concepts independent so you can use them for different output parameters.

### Tuning

| What to change | Where | Effect |
|---|---|---|
| **Dead zone size** | `STILL_THRESHOLD` | Larger = needs faster movement to register; smaller = more sensitive to slow motion |
| **Speed sensitivity** | Upper input in a future `map()` call | Lower = small movements feel fast; higher = needs big sweeps |
| **Read rate** | `READ_INTERVAL` | Faster = more responsive but noisier velocity; slower = smoother but delayed |
| **Add visual output** | Write a draw function using [Canvas](class06-CodePatterns.md#the-loop-pattern) or [Animation](class05-Feb06.md#creating-an-animation) mode | Connect velocity/direction to the screen — the workshop task |

---

## Going Further

- **Trigger animations on fast approach:** Use `if (direction == -1 && speed > 20)` to detect a quick hand swipe toward the sensor. Play an alert animation that wouldn't trigger from slow movement.
- **Use velocity for animation speed:** `screen.setSpeed(map(speed, 0, 50, 2.0, 0.1))` — faster hand movement = faster animation playback.
- **Add smoothing to velocity:** The velocity calculation can be jittery. Apply the same [rolling average technique](class07-pressure_smoothing.md) to the `velocity` variable for smoother motion detection.
- **Combine with pressure for multi-gesture input:** Distance detects approach, pressure detects contact. Sequence them: "fast approach → firm press" could trigger a different response than "slow approach → light press."
- **Track acceleration:** Store velocity from two consecutive frames and compute `acceleration = (currentVelocity - previousVelocity) / elapsed`. This tells you whether movement is speeding up or slowing down.

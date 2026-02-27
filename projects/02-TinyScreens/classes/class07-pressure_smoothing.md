# Pressure Sensor — Rolling Average Smoothing

[← Back to Class 07](class07-Feb27.md)

Raw `analogRead()` values jump around from frame to frame — even when you're holding the sensor perfectly still. This jitter makes your visual output look twitchy and unintentional. **Rolling average smoothing** fixes this by averaging the last N readings, producing a stable value that still responds to real changes.

---

## The Concept

Instead of using each raw reading directly, keep the last N readings in an array. Every time you read the sensor, add the new value to the array (replacing the oldest one) and average all N values. The result is a **smoothed** value that filters out noise.

```
Raw readings:    312  287  345  298  320  310  335  290  315  305
                  └────────────────────────────────────────────┘
                         Average of last 10 = ~312
                                                  ↓
Smoothed output:                                 312
```

The array works as a **circular buffer** — an index keeps track of where to write next, and when it reaches the end, it wraps back to the beginning:

```
Array:  [ 312  287  345  298  320  310  335  290  315  305 ]
Index:                                                  ↑
Next write goes here ─────────────────────────────────→ wraps to [0]
```

The trade-off: **larger window = smoother output, but slower response** to real changes. A window of 5 feels responsive; a window of 20 feels sluggish but very stable. Start with 10 and adjust.

---

## Complete Sketch

This sketch smooths the pressure sensor reading using a rolling average and prints the result to Serial Monitor — **no screen output is included**. In the [workshop](#putting-it-all-together), your group will add visual output.

```cpp
// ============================================================
// Pressure Sensor — Rolling Average Smoothing
// ============================================================
// What this does:
//   Stores the last WINDOW_SIZE sensor readings in an array.
//   Each frame, the oldest reading is replaced with the newest,
//   and the average of all readings becomes the smoothed value.
//   The smoothed value is printed to Serial Monitor for testing.
//
// Why this matters:
//   Raw analogRead() values jitter from frame to frame.
//   Smoothing filters out the noise so your visual output
//   feels intentional and controlled, not twitchy.
//
// Key concept:
//   Circular buffer — an array with a wrapping index
// ============================================================

// --- Pin configuration ---
int FSR_PIN = A0;

// --- Smoothing settings ---
// WINDOW_SIZE controls how many readings are averaged.
// Bigger = smoother but slower to respond.
// Smaller = more responsive but more jittery.
const int WINDOW_SIZE = 10;

int readings[WINDOW_SIZE];    // circular buffer of past readings
int bufferIndex = 0;          // where to write the next reading
long runningTotal = 0;        // sum of all values in the buffer

// --- Sensor range ---
// Use your observed min/max from Serial Monitor (or calibrated values)
int LIGHTEST_PRESS = 50;
int HARDEST_PRESS = 800;

// --- Timing ---
int READ_INTERVAL = 20;
unsigned long lastRead = 0;

// --- Values ---
int smoothedPressure = 0;     // the averaged sensor value

// ── Smoothing function ──
// Reads the sensor, updates the circular buffer, and returns
// the average of the last WINDOW_SIZE readings.
int readSmoothed(int pin)
{
    int newReading = analogRead(pin);

    // Subtract the oldest reading from the total
    runningTotal -= readings[bufferIndex];

    // Store the new reading in the buffer
    readings[bufferIndex] = newReading;

    // Add the new reading to the total
    runningTotal += newReading;

    // Advance the index, wrapping around at the end
    bufferIndex = (bufferIndex + 1) % WINDOW_SIZE;

    // Return the average
    return runningTotal / WINDOW_SIZE;
}

void setup()
{
    Serial.begin(9600);

    // Initialize the buffer with zeros
    for (int i = 0; i < WINDOW_SIZE; i++)
    {
        readings[i] = 0;
    }
}

void loop()
{
    if (millis() - lastRead >= READ_INTERVAL)
    {
        lastRead = millis();

        // READ + SMOOTH
        smoothedPressure = readSmoothed(FSR_PIN);

        // OUTPUT — print to Serial Monitor
        Serial.print("Smoothed: ");
        Serial.println(smoothedPressure);
    }
}
```

---

## What to Notice

- **The `readSmoothed()` function encapsulates all the buffer logic.** From the outside, it looks just like `analogRead()` — you call it with a pin and get back a number. But the number it returns is stable.
- **`runningTotal` avoids re-summing the whole array every frame.** Instead of adding up all N values each time, we subtract the oldest and add the newest. This is efficient and scales well to larger window sizes.
- **The circular buffer pattern** (`bufferIndex = (bufferIndex + 1) % WINDOW_SIZE`) is the same pattern used in the [Dot Trail example](class07-canvas_dot_trail.md). Same concept, different application.
- **Initializing the buffer to zeros** means the first few readings will be artificially low (averaged with zeros). This corrects itself after `WINDOW_SIZE` readings — usually within the first fraction of a second.
- **No screen output is included.** This sketch focuses purely on the smoothing technique. Your group will add visual output in the workshop.

### The Window Size Trade-off

| Window Size | Smoothness | Responsiveness | Best For |
|---|---|---|---|
| 3–5 | Low smoothing | Very responsive | Fast interactions, quick gestures |
| 8–12 | Moderate smoothing | Good balance | Most projects — start here |
| 15–25 | High smoothing | Noticeable lag | Slow, ambient interactions |

### Tuning

| What to change | Where | Effect |
|---|---|---|
| **Smoothness** | `WINDOW_SIZE` | Larger = smoother but slower response |
| **Sensor range** | `LIGHTEST_PRESS` / `HARDEST_PRESS` | Adjust to match your sensor's real range |
| **Add visual output** | Write a draw function using [Canvas](class06-CodePatterns.md#the-loop-pattern) or [Animation](class05-Feb06.md#creating-an-animation) mode | Connect this smoothed value to the screen — the workshop task |
| **Read rate** | `READ_INTERVAL` | Faster reads fill the buffer sooner but use more CPU |

---

## Going Further

- **Try different window sizes** — upload the sketch, change `WINDOW_SIZE` to 3, then 20, and feel the difference. This is the best way to understand the trade-off.
- **Apply to distance sensor too** — the same `readSmoothed()` pattern works with distance. Replace `analogRead(pin)` with `ultrasonic.getDistanceCM()` inside the function (and change the return type to `float`).
- **Combine with calibration** — smoothing handles per-frame noise; calibration handles per-build variation. See [Calibration + Smoothing Combined](class07-pressure_calibration_smoothing.md).
- **Use smoothed values for animation speed** — jittery speed changes look broken. Smoothed values make `screen.setSpeed()` feel intentional.

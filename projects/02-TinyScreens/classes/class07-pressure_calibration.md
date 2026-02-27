# Pressure Sensor — Auto-Calibration at Startup

[← Back to Class 07](class07-Feb27.md)

In [Class 06](class06-Feb13.md), you hard-coded your sensor range (`LIGHTEST_PRESS`, `HARDEST_PRESS`) based on Serial Monitor observations. That works for testing, but every custom FSR is different — and even the same sensor can read differently depending on temperature, humidity, or how the wires are seated today.

**Auto-calibration** solves this by measuring the sensor's resting state automatically during `setup()`. Thresholds are then set relative to that measured baseline, so the sketch adapts every time it powers on.

---

## The Concept

When nobody is pressing the sensor, `analogRead()` returns a **baseline** value. This isn't always 0 — it depends on your materials and wiring. Calibration reads the sensor many times at startup, averages the results, and stores that average as `baseline`.

From there, thresholds are defined as **offsets above baseline**:

```
baseline                  baseline + lightOffset        baseline + firmOffset
   ↓                            ↓                              ↓
   ├──── IDLE ──────────────────┼──── LIGHT PRESS ─────────────┼──── FIRM PRESS ────→
```

If the baseline shifts from 20 to 45 between power cycles, the thresholds shift with it — no code edits needed.

---

## Complete Sketch

This sketch calibrates at startup and classifies pressure into three zones (idle, light, firm). It prints values to Serial Monitor — **no screen output is included**. In the [workshop](#putting-it-all-together), your group will add visual output.

```cpp
// ============================================================
// Pressure Sensor — Auto-Calibration at Startup
// ============================================================
// What this does:
//   During setup(), reads the pressure sensor 50 times over
//   ~1 second to establish a resting baseline. Thresholds
//   for "light press" and "firm press" are set relative to
//   that baseline. In loop(), the sensor reading is classified
//   into a zone and printed to Serial Monitor.
//
// Why this matters:
//   Every custom FSR has a different resting value, and that
//   value can drift between sessions. Calibration means your
//   thresholds work every time without manual tuning.
//
// Key concept:
//   baseline + offset = threshold
// ============================================================

// --- Pin configuration ---
int FSR_PIN = A0;

// --- Calibration settings ---
// How many samples to take during calibration
// More samples = more accurate baseline, but longer startup
int CALIBRATION_SAMPLES = 50;
int CALIBRATION_DELAY = 20;       // ms between samples during calibration

// --- Threshold offsets ---
// These define how far ABOVE the baseline each zone starts.
// Tune these based on how your sensor feels.
// Smaller offsets = more sensitive; larger = requires harder press.
int LIGHT_PRESS_OFFSET = 30;     // baseline + this = start of "light press"
int FIRM_PRESS_OFFSET = 120;     // baseline + this = start of "firm press"

// --- Calibrated values (set during setup) ---
int baseline = 0;                // resting value — measured automatically
int lightPressThreshold = 0;     // baseline + LIGHT_PRESS_OFFSET
int firmPressThreshold = 0;      // baseline + FIRM_PRESS_OFFSET

// --- Timing ---
int READ_INTERVAL = 20;
unsigned long lastRead = 0;

// --- Sensor value ---
int gripStrength = 0;            // current pressure reading

// ── Calibration function ──
// Reads the sensor `samples` times, returns the average.
// Call this ONCE in setup() with the sensor at rest.
int calibrateSensor(int pin, int samples)
{
    long total = 0;

    for (int i = 0; i < samples; i++)
    {
        total += analogRead(pin);
        delay(CALIBRATION_DELAY);     // delay is OK here — we're in setup()
    }

    return total / samples;
}

// ── Read function ──
// Returns the current pressure reading
int readPressure()
{
    return analogRead(FSR_PIN);
}

// ── Classification function ──
// Returns which zone the current reading falls in:
// 0 = idle, 1 = light press, 2 = firm press
int classifyPress(int pressure)
{
    if (pressure >= firmPressThreshold)
    {
        return 2;  // firm press
    }
    else if (pressure >= lightPressThreshold)
    {
        return 1;  // light press
    }
    else
    {
        return 0;  // idle
    }
}

void setup()
{
    Serial.begin(9600);

    // --- Calibrate ---
    // Leave the sensor UNTOUCHED during startup.
    // The sketch reads it 50 times to find the resting baseline.
    Serial.println("Calibrating... do not touch the sensor.");

    baseline = calibrateSensor(FSR_PIN, CALIBRATION_SAMPLES);

    // Set thresholds relative to the measured baseline
    lightPressThreshold = baseline + LIGHT_PRESS_OFFSET;
    firmPressThreshold = baseline + FIRM_PRESS_OFFSET;

    // Print calibration results so you can verify
    Serial.print("Baseline: ");
    Serial.println(baseline);
    Serial.print("Light press threshold: ");
    Serial.println(lightPressThreshold);
    Serial.print("Firm press threshold: ");
    Serial.println(firmPressThreshold);
    Serial.println("Calibration complete. Ready.");
}

void loop()
{
    if (millis() - lastRead >= READ_INTERVAL)
    {
        lastRead = millis();

        // READ
        gripStrength = readPressure();

        // TRANSLATE
        int zone = classifyPress(gripStrength);

        // OUTPUT — print to Serial Monitor
        Serial.print("Grip: ");
        Serial.print(gripStrength);
        Serial.print(" | Zone: ");
        Serial.println(zone);
    }
}
```

---

## What to Notice

- **`delay()` in `setup()` is fine.** Calibration happens once before `loop()` starts. Using `delay()` here is simpler and doesn't affect responsiveness.
- **The offsets are the only values you tune.** `LIGHT_PRESS_OFFSET` and `FIRM_PRESS_OFFSET` define how sensitive each zone is. Start with the values above and adjust based on feel — if light press triggers too easily, increase the offset.
- **Serial output during calibration** lets you verify the baseline and thresholds every time you power on. If the numbers look wrong, check your wiring and sensor construction.
- **The classification function returns a number (0, 1, 2)** instead of a string. Numbers are easier to use in conditionals and `switch/case` statements.

### Tuning

| What to change | Where | Effect |
|---|---|---|
| **Sensitivity** | `LIGHT_PRESS_OFFSET` / `FIRM_PRESS_OFFSET` | Smaller offset = triggers with less pressure |
| **Number of zones** | Add more thresholds in `classifyPress()` | e.g., add `MEDIUM_PRESS_OFFSET` for a 4-zone system |
| **Calibration accuracy** | `CALIBRATION_SAMPLES` | More samples = more stable baseline, but longer startup |
| **Add visual output** | Write a `drawFeedback(zone)` function using [Canvas](class06-CodePatterns.md#the-loop-pattern) or [Animation](class05-Feb06.md#creating-an-animation) mode | Connect this input to the screen — the workshop task |

---

## Going Further

- **Add a recalibration trigger:** Wire a button and call `calibrateSensor()` again when pressed — useful if the sensor shifts during use.
- **Use calibrated thresholds with animation mode:** Replace `drawFeedback()` with `screen.play()` calls to switch animations per zone (see [Pressure Threshold → Animation](class06-pressure_threshold_animation.md) for the pattern).
- **Combine with `map()`:** Instead of zones, `map()` the reading from `baseline` to `baseline + FIRM_PRESS_OFFSET` for smooth continuous output — the baseline ensures the map starts at the right place.
- **Add smoothing:** Calibration fixes the *starting point*; smoothing fixes *jitter*. See [Rolling Average Smoothing](class07-pressure_smoothing.md) and [Calibration + Smoothing Combined](class07-pressure_calibration_smoothing.md).

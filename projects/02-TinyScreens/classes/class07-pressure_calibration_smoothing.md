# Pressure Sensor — Calibration + Smoothing Combined

[← Back to Class 07](class07-Feb27.md)

This sketch combines the two techniques from the previous pages:

- [Auto-Calibration at Startup](class07-pressure_calibration.md) — measures the sensor's resting baseline so thresholds adapt to your specific build
- [Rolling Average Smoothing](class07-pressure_smoothing.md) — averages the last N readings so the output is stable, not jittery

**Calibration** solves per-build variation (different baselines between sensors). **Smoothing** solves per-frame noise (jitter between consecutive readings). Together they give you the most reliable pressure input possible from a custom FSR.

---

## How They Work Together

```
Power on
   │
   ▼
┌──────────────────────┐
│  CALIBRATION (setup)  │   Read sensor 50× at rest → baseline
│  baseline = 42        │   lightThreshold = 42 + 30 = 72
│  lightThreshold = 72  │   firmThreshold  = 42 + 120 = 162
│  firmThreshold = 162  │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────────────────────────────────────────┐
│  SMOOTHING (every loop)                                   │
│                                                           │
│  Raw readings:  48  55  41  62  39  44  51  47  53  45    │
│                  └────────── average = 48.5 ──────────┘   │
│                                                           │
│  smoothedPressure = 48  →  zone = IDLE (below 72)         │
└──────────────────────────────────────────────────────────┘
```

Without calibration, you'd hard-code `LIGHTEST_PRESS = 50` and hope it's close. Without smoothing, the zone might flicker between IDLE and LIGHT on noisy readings near the threshold. With both, the baseline is measured and the readings are stable.

---

## Complete Sketch

This sketch calibrates at startup, smooths every reading, and classifies into three zones. It prints values to Serial Monitor — **no screen output is included**. In the [workshop](#putting-it-all-together), your group will add visual output.

```cpp
// ============================================================
// Pressure Sensor — Calibration + Smoothing Combined
// ============================================================
// What this does:
//   1. Calibrates during setup() to find the resting baseline
//   2. Smooths every reading using a rolling average
//   3. Classifies the smoothed value into zones using
//      calibrated thresholds
//   4. Prints all values to Serial Monitor for testing
//
// Why both:
//   Calibration handles per-build variation (different sensors
//   have different resting values). Smoothing handles per-frame
//   noise (raw readings jitter even when pressure is constant).
//   Together they make the input reliable and the output stable.
// ============================================================

// --- Pin configuration ---
int FSR_PIN = A0;

// --- Calibration settings ---
int CALIBRATION_SAMPLES = 50;
int CALIBRATION_DELAY = 20;       // ms between calibration reads

// --- Threshold offsets (above baseline) ---
// These define zone boundaries relative to the calibrated baseline.
// Tune these based on how your sensor feels when pressed.
int LIGHT_PRESS_OFFSET = 30;
int FIRM_PRESS_OFFSET = 120;

// --- Calibrated values (set during setup) ---
int baseline = 0;
int lightPressThreshold = 0;
int firmPressThreshold = 0;

// --- Smoothing settings ---
const int WINDOW_SIZE = 10;

int readings[WINDOW_SIZE];
int bufferIndex = 0;
long runningTotal = 0;

// --- Timing ---
int READ_INTERVAL = 20;
unsigned long lastRead = 0;

// --- Values ---
int smoothedPressure = 0;     // averaged sensor reading
int pressureZone = 0;         // 0 = idle, 1 = light, 2 = firm

// ── Calibration function ──
// Reads the sensor `samples` times at rest, returns the average.
int calibrateSensor(int pin, int samples)
{
    long total = 0;

    for (int i = 0; i < samples; i++)
    {
        total += analogRead(pin);
        delay(CALIBRATION_DELAY);
    }

    return total / samples;
}

// ── Smoothing function ──
// Reads the sensor, updates the circular buffer, returns the
// average of the last WINDOW_SIZE readings.
int readSmoothed(int pin)
{
    int newReading = analogRead(pin);

    runningTotal -= readings[bufferIndex];
    readings[bufferIndex] = newReading;
    runningTotal += newReading;

    bufferIndex = (bufferIndex + 1) % WINDOW_SIZE;

    return runningTotal / WINDOW_SIZE;
}

// ── Classification function ──
// Returns zone: 0 = idle, 1 = light press, 2 = firm press
int classifyPressure(int pressure)
{
    if (pressure >= firmPressThreshold)
    {
        return 2;
    }
    else if (pressure >= lightPressThreshold)
    {
        return 1;
    }
    else
    {
        return 0;
    }
}

void setup()
{
    Serial.begin(9600);

    // Initialize smoothing buffer
    for (int i = 0; i < WINDOW_SIZE; i++)
    {
        readings[i] = 0;
    }

    // --- Calibrate ---
    Serial.println("Calibrating... do not touch the sensor.");
    baseline = calibrateSensor(FSR_PIN, CALIBRATION_SAMPLES);

    lightPressThreshold = baseline + LIGHT_PRESS_OFFSET;
    firmPressThreshold = baseline + FIRM_PRESS_OFFSET;

    Serial.print("Baseline: ");
    Serial.println(baseline);
    Serial.print("Light threshold: ");
    Serial.println(lightPressThreshold);
    Serial.print("Firm threshold: ");
    Serial.println(firmPressThreshold);
    Serial.println("Ready.");

    // Pre-fill the smoothing buffer with baseline values
    // so the first few frames don't start from zero
    for (int i = 0; i < WINDOW_SIZE; i++)
    {
        readings[i] = baseline;
    }
    runningTotal = (long)baseline * WINDOW_SIZE;
}

void loop()
{
    if (millis() - lastRead >= READ_INTERVAL)
    {
        lastRead = millis();

        // READ + SMOOTH
        smoothedPressure = readSmoothed(FSR_PIN);

        // TRANSLATE — classify zone using calibrated thresholds
        pressureZone = classifyPressure(smoothedPressure);

        // OUTPUT — print to Serial Monitor
        Serial.print("Smoothed: ");
        Serial.print(smoothedPressure);
        Serial.print(" | Zone: ");
        Serial.println(pressureZone);
    }
}
```

---

## What to Notice

- **The smoothing buffer is pre-filled with `baseline` values** in `setup()`, after calibration. This prevents the first few smoothed readings from being artificially low (which would happen if the buffer started at zero).
- **`map()` uses `baseline` as the bottom of the input range**, not 0. This means the bar width maps from the resting state to the firm press threshold — all of the bar's visual range is used for the useful pressure range.
- **Both techniques use the same reading.** `readSmoothed()` calls `analogRead()` internally. The smoothed output feeds both `classifyPressure()` (for zone decisions) and any `map()` call you add later (for continuous output). You don't need to read the sensor twice.
- **No screen output is included.** This sketch focuses on reliable input. Your group will add visual output in the workshop — see [From Static to Dynamic](class06-CodePatterns.md#from-static-to-dynamic) for the step-by-step approach.

### Tuning

| What to change | Where | Effect |
|---|---|---|
| **Zone sensitivity** | `LIGHT_PRESS_OFFSET` / `FIRM_PRESS_OFFSET` | Smaller = easier to trigger; larger = needs more force |
| **Smoothness** | `WINDOW_SIZE` | Larger = smoother but slower; smaller = responsive but noisier |
| **Calibration accuracy** | `CALIBRATION_SAMPLES` | More samples = more stable baseline |
| **Add visual output** | Write a draw function using [Canvas](class06-CodePatterns.md#the-loop-pattern) or [Animation](class05-Feb06.md#creating-an-animation) mode | Connect this input to the screen — the workshop task |

---

## Going Further

- **Add a dead zone near baseline:** If the bar twitches slightly at idle, add a small gap: `if (smoothedPressure < baseline + 10) barWidth = 0;`. This ensures truly idle readings map to zero.
- **Use with animation mode:** Replace `drawPressureBar()` with `screen.play()` and `screen.setSpeed()` — use the smoothed value to control animation speed, and zones to switch between animations.
- **Add a recalibration button:** Read a digital pin; when pressed, re-run `calibrateSensor()` and update thresholds. Useful during exhibition if the sensor shifts.
- **Combine with distance sensor:** Use pressure for one visual parameter (size, intensity) and distance for another (position, speed). Each sensor can have its own smoothing buffer.

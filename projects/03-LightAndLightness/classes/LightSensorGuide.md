# Light Sensor — Photoresistor Setup Guide

[← Back to Light & Lightness](../LightAndLightness.md)

---

## Table of Contents

- [What is a Photoresistor?](#what-is-a-photoresistor)
- [How Does It Work?](#how-does-it-work)
- [How Does It Connect to Arduino?](#how-does-it-connect-to-arduino)
  - [The Voltage Divider Circuit](#the-voltage-divider-circuit)
  - [Wiring Diagram](#wiring-diagram)
- [How is the Data Read Within the Code?](#how-is-the-data-read-within-the-code)
  - [Reading Values](#reading-values)
  - [Understanding the Range](#understanding-the-range)
  - [Timing Considerations](#timing-considerations)
- [Examples](#examples)
- [Design Considerations](#design-considerations)

---

## What is a Photoresistor?

<!-- TODO: Add photo of photoresistor component -->

A photoresistor (also called a Light Dependent Resistor or LDR) is an analog sensor whose resistance changes based on the amount of light hitting its surface. More light = lower resistance. Less light = higher resistance.

Unlike the distance sensor used in Project 2 (which required a library), the photoresistor is read directly with `analogRead()` — the same function used for the pressure sensor. If you worked with the pressure sensor in Project 2, this will feel familiar.

---

## How Does It Work?

The photoresistor is made of a semiconductor material that becomes more conductive when exposed to light. As light intensity increases, its resistance drops, allowing more current to flow. This change in resistance is converted to a voltage that the Arduino can read.

<!-- TODO: Add diagram showing resistance vs. light relationship -->

---

## How Does It Connect to Arduino?

### The Voltage Divider Circuit

Because the Arduino reads **voltage** (not resistance), the photoresistor must be wired in a **voltage divider** circuit with a fixed resistor (typically 10kΩ). This converts the changing resistance into a changing voltage that `analogRead()` can measure.

<!-- TODO: Add voltage divider schematic diagram -->

| Component | Connection |
|-----------|------------|
| **Photoresistor — Leg 1** | 5V |
| **Photoresistor — Leg 2** | Analog Pin (e.g., A0) AND one leg of the 10kΩ resistor |
| **10kΩ Resistor — other leg** | GND |

### Wiring Diagram

<!-- TODO: Add breadboard wiring diagram image -->
<!-- TODO: Confirm which analog pin will be used in all code examples -->

---

## How is the Data Read Within the Code?

### Reading Values

No library is needed. Use Arduino's built-in `analogRead()`:

```cpp
int lightValue = analogRead(A0);
```

This returns a value between **0** (dark) and **1023** (bright).

### Understanding the Range

The raw range of 0–1023 depends heavily on your specific environment:

- **Indoor ambient light** might read 300–700
- **Covered / shadowed** might read 50–200
- **Direct light / flashlight** might read 800–1023

<!-- TODO: Add guidance on calibrating the range for a specific environment -->
<!-- TODO: Consider linking to a calibration code example -->

> **Tip:** Use the Serial Monitor to print your sensor values and understand the actual range in your working environment before mapping or setting thresholds.

### Timing Considerations

Unlike the ultrasonic distance sensor, the photoresistor can be read very quickly — there is no delay between readings. However, you may want to **smooth** your readings to avoid jitter:

<!-- TODO: Add simple rolling average code snippet -->
<!-- TODO: Consider cross-referencing the smoothing techniques from Project 2 (class07-pressure_smoothing.md) -->

---

## Examples

<!-- TODO: Link to workshop code examples as they are developed -->
<!-- TODO: Consider a basic "read and print to Serial Monitor" starter example -->

---

## Design Considerations

- **Orientation** — Where the sensor faces determines what light it reads. Pointing it toward a window creates very different behavior than pointing it at a desk lamp or away from direct sources.
- **Ambient Light** — The sensor reads ALL light in its field of view, not just a specific source. Room lighting, sunlight, and nearby screens all contribute. Consider how this environmental sensitivity becomes part of your object's behavior.
- **Shadow as Input** — Light sensors can also detect shadow. A hand or material passing between a light source and the sensor creates a readable change. This connects directly to the project's focus on light and shadow.
- **Sensor Placement** — Where the sensor sits on or within the object determines the interaction. Inside an enclosure, it may read the object's own shadow. Exposed, it reads the room. Consider this relationship carefully.
- **Sensitivity** — The 10kΩ resistor value affects sensitivity. A larger resistor makes the sensor more sensitive to low-light changes; a smaller one shifts sensitivity toward brighter conditions. The 10kΩ is a good general-purpose starting point.

<!-- TODO: Add notes about using the sensor with the object's own light/shadow output (feedback loops) -->

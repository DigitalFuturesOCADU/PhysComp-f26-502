# Class 10 | March 20 | Workshop 2

[← Back to Light & Lightness](../LightAndLightness.md)

---

## Overview

Integration: connect the light sensor to the servo so that light controls movement. You will build a working sensor-to-servo circuit and explore different ways to connect the two — continuous mapping, threshold switching, and timed moves triggered by light level.

## What We'll Cover

1. Wire a photoresistor and servo on the same breadboard
2. Map sensor values to servo position using `map()`
3. Use thresholds to switch between `oscillate()` behaviors
4. Use thresholds to trigger `moveServoA()` timed moves
5. Introduction to the "going further" guides for building more complex behaviors

## Resources

- [Sensor + Servo Guide](../SensorServoGuide.md) — overview of all patterns and techniques
- [Servo Movement Reference](../servo-movement-reference.md)
- [Light Sensor Guide](LightSensorGuide.md)
- [Delay vs Millis](../../01-AltController/DelayVsMillis.md)
- [Code Patterns (from Project 2)](../../02-TinyScreens/classes/class06-CodePatterns.md)

## Workshop

### Core Walkthrough

Work through the [Light Sensor to Servo — Step by Step](class10-LDR-to-Servo.md) guide. This takes you from a bare breadboard to a working sensor-driven servo in five stages:

1. **Wire and read the light sensor** — confirm values in the Serial Monitor
2. **Wire and test the servo** — confirm it moves to a position you set
3. **Connect the two with map()** — light controls position
4. **Threshold + oscillation** — different oscillation profiles for dark and bright
5. **Threshold + timed moves** — servo travels to different targets based on light level

Complete all five stages and upload each one to verify it works before moving on.

### Going Further

Once the core walkthrough is working, explore the guides below to expand your circuit and behaviors. These are independent — pick the one that is most relevant to your design idea:

- [Multiple Servos](MultipleServos.md) — add a second servo, run both independently
- [Multiple Light Sensors](MultipleLDRs.md) — add a second photoresistor, compare readings for directional sensing
- [Two LDRs + Two Servos](TwoLDRsTwoServos.md) — combinatory patterns for the full 2+2 setup
- [Physical Adjustments](PhysicalAdjustments.md) — extending wires to mount sensors and servos away from the breadboard
- [Light Sensor Guide — Smoothing](LightSensorGuide.md#3-smoothing--stabilizing-noisy-readings) — stabilize noisy readings
- [Light Sensor Guide — Calibration](LightSensorGuide.md#4-calibration--measuring-ambient-light-at-startup) — adapt to different rooms automatically

## Submission

- Working sketch with a light sensor controlling a servo — using `map()`, a threshold, or both
- Short video (15–30 seconds) showing the servo responding to changes in light
- Upload sketch file and video to Canvas

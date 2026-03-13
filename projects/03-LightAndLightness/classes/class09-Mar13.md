# Class 09 | March 13 | Workshop 1

[← Back to Light & Lightness](../LightAndLightness.md)

---

## Overview

This workshop is about getting the parts working. By the end of class you will have a light sensor (photoresistor) wired on a breadboard, reading values in the Serial Monitor, and a servo motor responding to those values. Nothing fancy — just the foundation.

### Building on Project 2

We are moving from screen-based output to material and movement, and from tactile input (pressure) to environmental input (light). The coding patterns carry over directly:

- `map()` and thresholds as a robust method for connecting input to output
- Data smoothing and filtering
- Managing multiple timings with a single loop

See [Code Patterns from Project 2](../../02-TinyScreens/classes/class06-CodePatterns.md) for a refresher.

---

## New Components

This is the first project where we use a **breadboard**, a **light-dependent resistor (LDR)**, and a **servo motor**. Below is a short introduction to each. External links are provided for deeper reading.

---

### Breadboard

![Breadboard layout](https://upload.wikimedia.org/wikipedia/commons/e/e8/Breadboard.png)

A breadboard is a prototyping tool that provides secure but temporary connections between wires and components — think of it as "connect the dots" for circuits.

- The **outer rails** run the length of the board and are used to distribute power (+ and −).
- The **inner columns** are connected in short rows across the center gap — each row of five holes is one electrical node.

This is where all of our circuits will be built for this project.

**Further reading:**
- [SparkFun — How to Use a Breadboard](https://learn.sparkfun.com/tutorials/how-to-use-a-breadboard/all)
- [Science Buddies — How to Use a Breadboard](https://www.sciencebuddies.org/science-fair-projects/references/how-to-use-a-breadboard)
- [Adafruit — Breadboards for Beginners (PDF)](https://cdn-learn.adafruit.com/downloads/pdf/breadboards-for-beginners.pdf)

---

### Light-Dependent Resistor (LDR)

![LDR wiring detail](../assets/1LDR_breadboard_detail.png)

A photoresistor (LDR) changes its resistance based on how much light hits it — more light means lower resistance. We read that change as a voltage on an analog pin, the same way we read the pressure sensor in Project 2.

#### LDR vs Pressure Sensor

| | Pressure Sensor (Project 2) | LDR / Photoresistor (Project 3) |
|---|---|---|
| **Wiring** | ![Pressure sensor wiring](../../02-TinyScreens/assets/pressureSensorWiring.png) | ![LDR wiring](../assets/1LDR_breadboard_bb.png) |
| **Input type** | Tactile (force) | Environmental (light) |
| **Read method** | `analogRead()` → 0–1023 | `analogRead()` → 0–1023 |
| **Resistor** | 220 Ω | 10 kΩ |
| **Why different?** | Lower resistance sensor needs a smaller pull-down | Higher resistance sensor needs a larger pull-down to create a readable voltage divider |

The code pattern is almost identical — only the resistor value and the physical input change.

**Further reading:**
- [Starting Electronics — Arduino LDR Tutorial](https://startingelectronics.org/beginners/arduino-tutorial-for-beginners/arduino-LDR-tutorial/)
- [Circuit Basics — Pairing an LDR with Arduino](https://www.circuitbasics.com/pairing-a-light-dependent-resistor-ldr-with-an-arduino-uno/)
- [ArduinoYard — LDR with Arduino Complete Guide](https://arduinoyard.com/ldr-with-arduino/)

---

### Servo Motor

![Servo motor pins](../assets/servoPins.jpg)

A servo motor rotates to a specific **angle** (0–180°) and holds that position. This is different from a standard DC motor, which just spins at a speed in a direction. We are using **standard hobby servos** (not continuous-rotation servos).

The three wires connect as follows:

| Wire Colour | Connection |
|---|---|
| **Brown** | GND |
| **Red** | 5 V |
| **Orange** | Signal (pin 9) |

![Servo connected with jumper cables](../assets/servoPinsWjumpers.jpg)

**Further reading:**
- [Arduino Docs — Basic Servo Control](https://docs.arduino.cc/tutorials/generic/basic-servo-control/)
- [Circuit Basics — How to Control Servos with Arduino](https://www.circuitbasics.com/controlling-servo-motors-with-arduino/)
- [Last Minute Engineers — Servo Motor Arduino Tutorial](https://lastminuteengineers.com/servo-motor-arduino-tutorial/)
- [Servo Movement Reference (local)](../servo-movement-reference.md)

---



## Workshop 1 : Prototype V1 and Design V1

All workshops in Project 3 have two components: a **working prototype** and a **design sketch**. Over the three workshops you will develop both.

### Working Prototype 1 — Control servo rotation with light

The goal is the simplest possible input → output connection between the sensor and the servo. Follow the step-by-step guide above so that the light sensor value controls the servo rotation in real time. The servo horn must have an additional object attached to it.

Follow the step-by-step guide:

**[Light Sensor to Servo — Step by Step](class09-LDR-to-Servo.md)**


### Project Design V1

In parallel, create a drawing that shows your initial concept. Read the [project brief](../LightAndLightness.md) carefully — yes, it is early, but you will improve the design each week through iteration. The drawing should note:

1. General concept of the interaction — how is light being used to affect movement?
2. Position of the light sensor
3. Servo motor(s) — what are they attached to and what do they do?
4. Overall form factor and size — on a table, free-standing, hanging, etc.
5. How is it being powered?

---

## Submission

Create a post in the [Workshop 1 Discussion](https://canvascloud.ocadu.ca/courses/13538/discussion_topics/212719) that includes:

- A video of the interaction between the light sensor and the servo
- An image of V1 of your design
- 1–2 sentence description of your design concept

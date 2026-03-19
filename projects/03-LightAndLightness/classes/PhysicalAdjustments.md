# Physical Adjustments — Extending Wires

[← Back to Sensor + Servo Guide](../SensorServoGuide.md) · [← Back to Light & Lightness](../LightAndLightness.md)

---

Sensors and servos connect to the Arduino with short jumper wires. For most designs, you will want to mount them away from the breadboard — on a wall, inside an enclosure, at the end of an arm, or embedded in a material. This guide covers the practical techniques for extending those connections.

---

## Why Extend?

The standard jumper wires in your kit are about 10–15 cm long. That is enough for testing on the bench, but most physical designs need:

- The **servo** mounted inside a mechanism or on a surface, away from the breadboard
- The **light sensor** positioned where it can read the light environment you care about — facing outward, pointing at a wall, inside an enclosure, or on the opposite side of your object from the Arduino

Longer wires give you freedom to separate the electronics from the physical form.

---

## What You Need

- **Solid core wire** — available in the XFab (fabrication lab). Solid core holds its shape when bent and solders easily. Ask for 22-gauge or 24-gauge.
- **Soldering iron and solder** — available in the XFab. You will solder wire to the photoresistor legs.
- **Pin-to-pin jumper cables** — from your kit. Used to chain connections for the servo.
- **Heat shrink tubing or electrical tape** — to insulate solder joints (optional but recommended).

---

## Extending Servo Wires

The servo plug connects to the breadboard through pin-to-pin jumper cables. To extend the reach, chain additional jumper cables end to end, or replace the jumper cables entirely with longer solid core wire.

### Option A — Chain Jumper Cables

Connect two or three pin-to-pin jumper cables end to end. Plug one end into the breadboard and the other into the servo's jumper cable. Each joint is a male-to-male connection.

**Advantages:** No soldering. Easy to adjust length. Uses parts you already have.

**Disadvantages:** Each connection is a potential point of failure. Long chains can pull apart. Signal quality is fine for servo PWM over reasonable distances (under 1 meter).

### Option B — Solid Core Wire

Cut three lengths of solid core wire (one for signal, one for 5V, one for GND). Strip both ends. Push one end into the breadboard and connect the other end to the servo's jumper cable.

**Advantages:** Cleaner, more reliable. Solid core holds its shape and can be routed along surfaces or through holes.

**Disadvantages:** Requires cutting and stripping wire. Still needs the jumper cable connection at the servo end (the servo plug does not accept bare wire directly).

> **Tip:** Keep the three wires together — bundle them with a small piece of tape every few inches. This prevents tangling and makes it easier to route the connection through your design.

---

## Extending Light Sensor Wires

The photoresistor has two short metal legs that push into the breadboard. To mount it away from the board, solder longer wires directly to those legs.

### Soldering Steps

1. **Cut two lengths of solid core wire** to the length you need. Strip about 1 cm of insulation from each end.
2. **Wrap** the stripped end of each wire around one leg of the photoresistor. One full wrap is enough — it holds the wire in place while you solder.
3. **Solder** the joint. Touch the soldering iron to both the wire and the leg simultaneously, then feed solder into the joint. A good joint is shiny and smooth, not blobby.
4. **Insulate** each joint with heat shrink tubing or a small piece of electrical tape so the two connections cannot touch each other.
5. **Push the free ends** of the solid core wire into the breadboard in the same positions the photoresistor legs would have occupied — one to the 5V row (through the 10kΩ resistor connection), one to the analog pin row.

The 10kΩ resistor stays on the breadboard. Only the photoresistor moves to a remote location. The voltage divider circuit is the same — you have just made two of the wires longer.

> **Tip:** Solid core wire holds its shape, so you can bend it to point the sensor in a specific direction and it will stay. This is useful for aiming the sensor at a particular light source or mounting it at an angle inside an enclosure.

---

## Wiring Diagram — Extended Setup

This diagram shows two LDRs with extended wires and two servos:

![Two LDRs with long wires and two servos on a breadboard](../assets/2LDR_LONG_2Servo_breadboard_bb.png)

The photoresistors are shown at the ends of their extended wires, away from the breadboard. The circuit is electrically identical to the standard setup — only the wire length has changed.

---

## Practical Notes

- **Wire length** — For servo signals, runs up to about 1 meter work fine. For the LDR voltage divider, longer wires add a tiny amount of resistance but this is negligible for the distances you will use in this project (under 2 meters). If you notice reading instability with very long runs, add a smoothing function. See [Light Sensor Guide — Smoothing](LightSensorGuide.md#3-smoothing--stabilizing-noisy-readings).

- **Strain relief** — Where the wire exits the breadboard or enters your physical form, secure it so that tugging on the wire does not pull the connection out of the breadboard. A small loop of tape at the exit point works.

- **Testing** — Always test the extended connection before building it into your design. Read the sensor values in the Serial Monitor and move the servo through its range to confirm everything works at the new distance.

- **Labeling** — When you have four or more wires running from the breadboard to remote components, label each one at both ends (a small piece of masking tape with "Servo A signal" or "LDR B" is enough). This saves significant debugging time.

### Reference

- [Multiple Servos](MultipleServos.md) — wiring and code for two servos
- [Multiple Light Sensors](MultipleLDRs.md) — wiring and code for two LDRs
- [Sensor + Servo Guide](../SensorServoGuide.md) — overview of all topics

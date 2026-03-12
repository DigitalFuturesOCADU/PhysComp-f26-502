# Light & Lightness — Canvas HTML Page Guide

This document maps the content for the Canvas LMS project page (`LightAndLightness.html`). It follows the same HTML structure and styling as the [Tiny Screens Canvas page](tinyScreenProjectPage.html). Each section below corresponds to an HTML `<div>` block in the page.

Use this as a checklist when building the final HTML. Content marked with `<!-- TODO -->` still needs to be written or confirmed before the page is published.

---

## 1. Main Image

**HTML element:** Full-width `<img>` in a `<div>` with `margin-bottom: 24px`

**Source reference:** tinyScreenProjectPage.html uses a Canvas-hosted image via `canvascloud.ocadu.ca/courses/13538/files/...`

| Field | Value |
|---|---|
| Image file | `Light_Lightness_mainImage.jpg` (already uploaded to Canvas) |
| Alt text | `Light_Lightness_mainImage.jpg` |
| Canvas file ID | `4413622` |
| API endpoint | `https://canvascloud.ocadu.ca/api/v1/courses/13538/files/4413622` |

**Status:** Already in current `LightAndLightness.html` — no changes needed.

---

## 2. Metadata Table

**HTML element:** `<table>` with rows of key/value pairs, no border, `#fafafa` background

**Rows to include:**

| Key | Value | Status |
|---|---|---|
| Keywords | Servos, Mechanisms, Light Sensors, Kinetic Objects | Ready |
| Format | Individual | Ready |
| Project Examples | [Light & Lightness Github](https://github.com/DigitalFuturesOCADU/PhysComp-f26-502/blob/main/projects/03-LightAndLightness/LightAndLightness.md) | Ready |
| Late Work Policy | Link to Canvas Late Work Policy page | Ready (same link as P2) |
| Resources | Servo Movement Reference, Light Sensor Guide, Delay vs Millis | Ready (linked in HTML) |

---

## 3. Deliverables

**HTML element:** Bordered card with "Deliverables" header, containing a date/task table

**Rows to include:**

| Date | Deliverable | Canvas Link | Status |
|---|---|---|---|
| March 13 | Workshop 1 | https://canvascloud.ocadu.ca/courses/13538/assignments/108893 | Ready (linked in HTML) |
| March 20 | Workshop 2 | https://canvascloud.ocadu.ca/courses/13538/assignments/108894 | Ready (linked in HTML) |
| March 27 | Workshop 3 | https://canvascloud.ocadu.ca/courses/13538/assignments/108895 | Ready (linked in HTML) |
| April 3 | Project Exhibition | `https://canvascloud.ocadu.ca/courses/13538/assignments/108818` | Ready (already linked in current HTML) |
| April 8 | Documentation Due on Website | `https://canvascloud.ocadu.ca/courses/13538/assignments/108817` | Ready (already linked in current HTML) |

---

## 4. Introduction

**HTML element:** Bordered card with "Introduction" header (matching P2's naming), two `<p>` elements

**Content (from LightAndLightness.md Project Overview):**

> **Paragraph 1:** This project focuses on creating responsive kinetic objects that manipulate light and shadow through movement. You will use servo motors to control mechanisms built primarily from lightweight materials — paper, vellum, fabric — and read input from a light sensor to affect and change its behavior over time. You should consider its behavior for the entire 30 minute span of the open exhibition. How can it adapt and change over that time? How can you accentuate and expand the physical and visual properties of your chosen material by controlling it with the servo?

> **Paragraph 2:** The input to your design will come from 1 or more light sensors. These sensors are highly versatile and can be used to read a range of interactions based on their location and orientation. Consider how to embed them into your overall design in a way that focuses the interaction between people, space, light, and the object itself.

**Status:** Updated in HTML — matches above.

---

## 5. From Screen to Space : Changes and Similarities from Project 2

**HTML element:** Bordered card with "From Screen to Space : Changes and Similarities from Project 2" header

**This section is NEW — P2 does not have an equivalent.** It bridges P2→P3 concepts for students.

**Content (condensed from LightAndLightness.md):**

Subsections to include as `<p>` blocks with bold labels:

- **Physical Output vs. Screen Output** — Servo movement in 3D space instead of LED matrix pixels. New concerns: torque, range of motion, speed, sound. Also new opportunities for the relationship between object, viewer, and light source.
- **Analog Input: Light vs. Pressure** — Both use `analogRead()` and return 0–1023. Pressure was direct and gestural; light is environmental and ambient. Interaction can be subtle and indirect or require active engagement. Must consider feedback loop of object manipulating light hitting sensor.
- **Mapping and Thresholds** — Same `map()` and threshold concepts. Now mapping to servo angles, sweep speeds, and motion ranges. We will use these core interaction concepts again, looking at how they differ based on input and output type.
- **Timing** — Controlling the timing of reading input and the speed of output is critical. `millis()` remains an important tool for developing a responsive interactive system that updates data at different rates.

**Status:** Updated in HTML — matches above.

---

## 6. Movement as Material

**HTML element:** Bordered card with "Movement as Material" header

**This is the P3 equivalent of P2's "Interaction as Preposition" section.**

**Content (condensed from LightAndLightness.md):**

- **Opening paragraph:** Movement is a design material — its speed, rhythm, range, and character communicate meaning. A servo rotates back and forth. What you build with it through code, mechanism, and material defines your piece's character.

- **Qualities of Movement table:**

| Quality | Description |
|---|---|
| Speed | fast / slow / accelerating / decelerating |
| Range | wide sweep / narrow wobble |
| Rhythm | continuous / intermittent / syncopated |
| Character | smooth / jerky / hesitant / aggressive |
| Responsiveness | immediate / gradual / delayed / accumulating |

- **Light and Shadow as Feedback:** The object casts shadows, blocks light, creates patterns. Consider: does the shadow tell a different story? Does the object's own movement change the light hitting its sensor (feedback loop)?

- **Lightness of Materials:** Physical lightness — paper/vellum/fabric amplify small servo movements. Optical lightness — these materials interact with light (translucent, reflective, diffusing).

- **Mechanisms:** Mechanisms are another key tool in shaping the character of motion. Servos output rotational movement, but this can be amplified or altered through mechanisms. Well-designed mechanisms provide opportunities that aren't possible with code alone, such as converting rotational movement into linear.

<!-- TODO: Decide if the Qualities of Movement table should include the "Code Lever" column from the markdown version or keep it simpler for the Canvas audience. -->
<!-- TODO: Add reference images for kinetic shadow art if available. -->

---

## 7. Hardware

**HTML element:** Bordered card with "Hardware" header (or "Tools and Components" to match current HTML naming)

**Content:**

**Input:**
- Light Sensor (Photoresistor) — Analog sensor read with `analogRead()`. Wired in a voltage divider with a 10kΩ resistor. Returns 0 (dark) to 1023 (bright). We will also be investigating how the possibilities of where the sensors are placed can be expanded by soldering longer wires to it. See: Light Sensor Guide for full details.

**Output:**
- Minimum: one or more servo motors. We will be focused on the Micro Servos included in your kit. These servos are small and are not particularly powerful, which is why the project focuses on manipulating lightweight materials. Work within the constraints of what the motors can do. See: Servo Movement Reference.
- *Optional* LEDs or other light sources — The primary output should come from the servos, but you do have the option of also using external LEDs or the LED Matrix.

<!-- TODO: Confirm whether to name this section "Hardware" (matching P2 GitHub) or "Tools and Components" (matching current P3 HTML). Recommend "Hardware" for consistency with P2 Canvas page. -->

---

## 8. Coding Concepts

**HTML element:** Bordered card with "Coding Concepts" header. Each concept is a bold label `<p>` followed by a description `<p>`.

**Concepts to include:**

- **Servo Movement Functions** — Rather than working with a complicated library, you will build the servo response using simple functions that can be layered and expanded to create complex movements. Full documentation in the Servo Movement Reference.

- **Reading a Light Sensor** — Use `analogRead()` to get values 0–1023. Use Serial Monitor to understand your range before mapping. We will also examine how concepts of smoothing, calibration, and relative data apply to light data.

- **Mapping Sensor Data to Movement** — Use `map()` to translate light values into movement parameters: servo angles, sweep speed, range of motion.

- **Thresholds** — Break light sensor range into zones (dark/medium/bright) that trigger different movement behaviors. Use `if/else` or `switch/case`.

- **State Machines** — Manage movement sequences and phases using `switch/case` with `moveServo()` completion checks. Light sensor input drives state transitions.

- **Timing without delay()** — Both movement functions use `millis()` internally. Never use `delay()`. See: Delay vs Millis.

<!-- TODO: Decide whether to include code snippets in the Canvas HTML. The P2 Canvas page did NOT include code — it linked to Arduino docs. The GitHub markdown has full snippets. Recommend keeping Canvas concise and linking to GitHub for code. -->
<!-- TODO: Add links — Arduino analogRead() docs, map() docs, if/else docs, switch/case docs, Delay vs Millis GitHub link, Servo Movement Reference GitHub link -->

---

## 9. Iterative Prototyping

**HTML element:** Bordered card with "Iterative Prototyping" header

**This section is NEW — P2 does not have an equivalent on the Canvas page.** It describes the week-by-week build arc.

**Content:**

- **Opening paragraph:** This project uses a deliberate iterative process to evolve your ideas over the 4 weeks. Starting with class 09, you will create your first concept drawings for the project alongside a working interaction between the sensor and the servos. From there you will be asked to iterate and update that design each week, meaning that what is shown at the exhibition is *at least* version 4. This method of development is standard practice within Physical Computing as a means to create refined results.

**Status:** Updated in HTML as a single paragraph — no per-workshop breakdown on Canvas page.

<!-- TODO: Decide if this section belongs on the Canvas page or should be deferred to the GitHub markdown and class pages. The P2 Canvas page did not have a prototyping arc section — the workshops were just listed in Deliverables. Including it would be new and potentially valuable for setting student expectations. -->

---

## 10. Design Constraints / Requirements

**HTML element:** Bordered card with "Design Constraints / Requirements" header. Contains a requirements `<ul>` followed by a "Do Not:" label and a `<ul>` of prohibitions.

**Requirements:**

- Primary materials for moving elements must be lightweight: paper, vellum, fabric, thread, wire, or similar
- Must manipulate light and shadow — movement creates visible changes in light/shadow
- Must use a light sensor as input — sensor reading must affect movement behavior over time
- Create unconventional kinetic movements — do not leave the servo as a simple untransformed sweep
- Consider the mechanical strength and limitations of servo motors
- All electronics and wiring enclosed and managed
- Write your own code — use the movement functions as a starting point, not a final product

**Do Not:**

- Use heavy or rigid materials (wood, acrylic, metal) as primary moving elements
- Leave the servo doing a simple untransformed sweep
- Ignore the light sensor in the final piece
- Glue your enclosure shut — allow access for maintenance
- Use `delay()` in your code
- Create an object that is based around stuffed animals
- Create an object that is based around a weapon

<!-- TODO: Review and expand the Do Not list — are there common student mistakes to preempt? Consider: don't attach materials that could jam the servo, don't build too large for the servo's torque, etc. -->

---

## 11. Design Considerations

**HTML element:** Bordered card with "Design Considerations" header. Each consideration is a bold label `<p>` followed by a description `<p>` (same format as P2).

**Considerations to include:**

- **Mechanism:** How does rotational servo motion become your kinetic movement? Consider linkages, cams, cranks, pulleys, levers, or direct drive. The mechanism is the design.

- **Responsiveness:** How does the object's behavior change with light? Gradual or sudden? Predictable or surprising? More active in light or darkness?

- **Materiality:** How do paper, vellum, or fabric amplify the servo's small rotational movement? A 30° sweep could make a sheet of paper billow across a wide arc.

- **Light & Shadow:** Is light the subject or the medium? Are you shaping shadow? Filtering light? Where is your light source, and how do your materials interact with it?

- **Craft & Finish:** Electronics enclosed, wires managed, materials attached with care, mechanism works reliably. Every choice should feel deliberate.

<!-- TODO: Consider adding an "Exhibition" consideration similar to P2's (P2 says: "Each work will display its chosen preposition"). What is the equivalent display/label strategy for P3? -->

---

## HTML Build Checklist

- [ ] Main image block (already in place)
- [ ] Metadata table with Resources row updated
- [ ] Deliverables table with Canvas assignment links
- [ ] Introduction card
- [ ] From Screen to Space card (new)
- [ ] Movement as Material card (new)
- [ ] Hardware card
- [ ] Coding Concepts card with external links
- [ ] Iterative Prototyping card (new — decide inclusion)
- [ ] Design Constraints / Requirements card with Do Not list
- [ ] Design Considerations card
- [ ] Final review: all Canvas assignment links working, all external links tested

<!-- TODO: Decide which new sections (From Screen to Space, Movement as Material, Iterative Prototyping) to include on the Canvas page vs. keeping only on GitHub. The GitHub md is the detailed reference; the Canvas page is the student-facing brief. P2's Canvas page matched its GitHub md closely. Recommend including all three for consistency. -->

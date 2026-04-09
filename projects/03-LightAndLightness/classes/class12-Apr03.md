# What's Next — Continuing Beyond the Course

[← Back to Light & Lightness](../LightAndLightness.md)

---

## Overview

You now have a working foundation in physical computing. Over three projects you went from basic digital input to analog sensors to servo-driven kinetic objects — building up a core set of skills along the way:

- Reading digital and analog sensors
- Writing code that responds to input in real time
- Using `millis()` for non-blocking timing
- Mapping sensor values to output ranges
- Controlling actuators (LEDs, servos)
- Soldering and physical prototyping
- Designing interactions between people, objects, and environments

These are not project-specific skills. They transfer directly to any physical computing work you do next — different sensors, different actuators, different platforms. The logic is the same. What changes is the range of components available to you and the complexity of what you build with them.

---

## Where to Go from Here

### Learn More Arduino

The best next step is to keep building. The Arduino ecosystem is enormous, and the official site is the starting point for everything:

- [Arduino](https://www.arduino.cc/) — official documentation, tutorials, language reference, and project ideas. The [Arduino Docs](https://docs.arduino.cc/) section is especially useful as a reference when you are working with a new board or library.

- [The Arduino Library](https://thearduinolibrary.com/) — a curated directory of Arduino libraries organized by category. Browsing libraries is one of the best ways to discover new methods and capabilities you did not know existed — each library page shows what functions are available and links to examples. If you are wondering "can Arduino do X?", searching here is a good place to start.

The board you used in this course (the Arduino UNO R4 WiFi) is just one option. Arduino makes boards with built-in WiFi, Bluetooth, more analog pins, faster processors, and smaller form factors. When your project outgrows the UNO, the answer is usually a different board — not a different platform.

### Component Suppliers and Learning Platforms

These companies sell sensors, actuators, and breakout boards — but more importantly, most of them also publish detailed tutorials, wiring diagrams, and example code for everything they sell. When you buy a sensor from one of these sites, the product page usually links directly to a getting-started guide. That combination of hardware + documentation makes them excellent learning resources, not just stores.

- [Adafruit](https://www.adafruit.com/) — excellent learning guides for every product, beginner-friendly documentation, and a strong community. Their [Learn System](https://learn.adafruit.com/) is one of the best free electronics education resources available. They also develop CircuitPython, an alternative to Arduino that lets you program microcontrollers in Python.

- [SparkFun](https://www.sparkfun.com/) — similar to Adafruit in quality and documentation. Their [Learn Tutorials](https://learn.sparkfun.com/tutorials) cover everything from basic circuits to advanced topics like GPS, IMUs, and wireless communication. Good place to explore sensors beyond what we used in class.

- [Seeed Studio](https://www.seeedstudio.com/) — known for their Grove system, which uses standardized connectors to eliminate breadboard wiring. Good option when you want to prototype quickly without worrying about wiring mistakes. They also make the XIAO series — very small, affordable microcontrollers.

- [DFRobot](https://www.dfrobot.com/) — wide range of sensors and actuators with Arduino-compatible libraries. Their [Learning Resources](https://www.dfrobot.com/blog) include project tutorials and component guides. Known for affordable sensor kits that cover a broad range of input types.

- [Makerfabs](https://www.makerfabs.com/) — focuses on ESP32-based boards and specialized modules (e-paper displays, LoRa radio, touch screens). Good resource when you want to explore connected devices and IoT projects.

### Beyond Arduino

Once you are comfortable with Arduino, you may want to explore platforms that offer more processing power, connectivity, or different programming languages.

It is also worth knowing that microcontroller boards come in many different form factors. The UNO is large and breadboard-friendly, but most projects beyond the prototyping stage benefit from something smaller, more portable, or more specialized:

- [Arduino Nano](https://store.arduino.cc/collections/nano-family) — the same ATmega chip family as the UNO, but in a much smaller package that fits directly into a breadboard. Good for space-constrained projects.
- [Adafruit Feather](https://www.adafruit.com/category/777) — a compact board format with built-in battery charging. Available in many variants (WiFi, Bluetooth, LoRa, etc.) and compatible with a large ecosystem of stackable add-on boards called Wings.
- [Adafruit Circuit Playground](https://www.adafruit.com/category/888) — a round board with built-in sensors (accelerometer, light, temperature, sound), NeoPixel LEDs, buttons, and a speaker. No wiring needed to get started — great for wearables, costumes, and quick interactive prototypes.
- [Adafruit QT Py](https://www.adafruit.com/category/595) — extremely small boards (thumbnail-sized) with USB-C and a Stemma QT connector for easy I2C wiring. Available in multiple chip variants. Good when you need a tiny brain for a project.
- [Seeed XIAO](https://www.seeedstudio.com/xiao-series-page) — similar in size to the QT Py, with variants based on different processors (SAMD21, RP2040, ESP32). Very affordable and compact.
- [ESP32 boards](https://www.espressif.com/en/products/socs/esp32) — powerful and inexpensive microcontrollers with built-in WiFi and Bluetooth. Available from many manufacturers in many form factors, including boards with built-in OLED or TFT screens like the [LilyGO T-Display](https://www.lilygo.cc/products/t-display-s3) and [Heltec WiFi Kit](https://heltec.org/project/wifi-kit-32-v3/). Popular for IoT and connected projects.

All of these boards can be programmed with the Arduino IDE using the same language and patterns you already know. The skills transfer directly — what changes is the size, power, connectivity, and built-in features of the board.

- [Raspberry Pi](https://www.raspberrypi.org/) — a full Linux computer the size of a credit card. Useful when your project needs image processing, machine learning, networking, or a graphical interface. Programs in Python, and can also run Arduino-style GPIO code. The [Raspberry Pi Pico](https://www.raspberrypi.com/products/raspberry-pi-pico/) is their microcontroller board — closer to an Arduino in function, but programmable in Python.

- [CircuitPython](https://circuitpython.org/) — developed by Adafruit, this lets you program microcontrollers using Python instead of C++. The syntax is more forgiving, and the workflow is simpler — you edit code files directly on the board like a USB drive. A good bridge if you already know some Python.

- [micro:bit](https://microbit.org/) — designed for education, with a built-in LED matrix, accelerometer, compass, and Bluetooth. Programs with block-based editors or Python. Useful for very quick prototyping and teaching.

---

## Sensors and Actuators to Explore

In this course you worked with buttons, pressure sensors, distance sensors, light sensors, LEDs, and servos. That is a small slice of what is available. Here are some categories worth exploring — each one uses the same fundamental patterns (read input, process, drive output) that you already know:

### More Input

| Sensor Type | What It Reads | Connection |
|---|---|---|
| Accelerometer / Gyroscope (IMU) | Orientation, tilt, motion, rotation | I2C or SPI |
| Temperature / Humidity | Environmental conditions | Analog or I2C |
| Sound / Microphone | Volume level, frequency, clap detection | Analog |
| Flex Sensor | Bending angle | Analog (voltage divider, like the LDR) |
| Capacitive Touch | Proximity or touch without mechanical contact | Digital or I2C |
| IR Receiver | Signals from remote controls | Digital |
| Color Sensor | RGB color values of a surface | I2C |
| Load Cell | Weight / force | Amplifier + analog |
| Encoder | Precise rotation position and direction | Digital (interrupts) |

### More Output

| Actuator Type | What It Does | Notes |
|---|---|---|
| Stepper Motor | Precise rotation in discrete steps | More control than a servo, can rotate continuously |
| DC Motor | Continuous rotation at variable speed | Needs a motor driver (H-bridge) |
| Solenoid | Linear push/pull motion | Good for striking, tapping, or latching |
| Relay | Switches high-power devices on/off | Lets Arduino control lamps, fans, appliances |
| Neopixel / Addressable LEDs | Individual color control per LED | One data pin controls hundreds of LEDs |
| Haptic Motor | Vibration feedback | Common in phones and game controllers |
| Piezo Buzzer / Speaker | Sound output, tones, melodies | Simple `tone()` function in Arduino |
| E-Paper Display | Low-power screen that holds image without power | Good for signage and slow-updating displays |

You will notice that many of these sensors connect via **I2C** or **SPI** — these are communication protocols that let multiple devices share just a few pins. Learning I2C is a natural next step and opens up a huge range of components.

### Qwiic / Stemma QT — Plug-and-Play I2C

The Arduino UNO R4 WiFi you used in this course has a **Qwiic connector** built into the board — the small connector near the corner labeled "QWIIC." This is a standardized I2C connector system developed by SparkFun (called **Qwiic**) and adopted by Adafruit under the name **Stemma QT**. The two are fully compatible — same connector, same pinout, same cables.

Instead of wiring SDA, SCL, power, and ground to a breadboard, you just plug in a small cable and the sensor is connected. You can also daisy-chain multiple sensors together on the same cable. This eliminates most wiring mistakes and makes it fast to try new sensors.

Both SparkFun and Adafruit sell large catalogs of Qwiic / Stemma QT compatible boards:

- [SparkFun Qwiic Ecosystem](https://www.sparkfun.com/qwiic) — accelerometers, environmental sensors, OLED displays, GPS modules, motor drivers, and more. Each product page includes an Arduino library and hookup guide.
- [Adafruit Stemma QT Sensors](https://www.adafruit.com/category/1005) — similar range of I2C sensors and breakouts. Adafruit's learn guides walk through wiring, library installation, and example code for each board.

If you want to explore new sensors with minimal wiring, start here. Pick a Qwiic/Stemma QT sensor, plug it into your R4 WiFi, install the library, and run the example sketch.

---

## Project Inspiration and Community

- [Hackaday](https://hackaday.com/) — daily coverage of hardware projects, electronics art, and creative engineering. The [Hackaday.io](https://hackaday.io/) project hosting site has thousands of documented builds.

- [Instructables](https://www.instructables.com/circuits/) — step-by-step project guides with photos. Quality varies, but the best ones are thorough and well-documented. Good for finding a project that matches your skill level and interests.

- [Arduino Project Hub](https://projecthub.arduino.cc/) — community-submitted Arduino projects with code, wiring diagrams, and instructions.

- [Make: Magazine / Makezine](https://makezine.com/) — the original maker publication. Covers electronics, fabrication, craft, and DIY culture.

- [Reddit r/arduino](https://www.reddit.com/r/arduino/) and [r/electronics](https://www.reddit.com/r/electronics/) — active communities for troubleshooting, project sharing, and learning.

---

## Local Resources (Toronto)

- **OCAD XFab** — you already know this one. Continue using the fabrication lab for laser cutting, 3D printing, and electronics work while you have access.
- [Toronto Tool Library](https://torontotoollibrary.com/) — lending library for tools including electronics equipment
- [Hacklab.TO](https://hacklab.to/) — Toronto's longest-running hackerspace. Regular open nights, member workshops, and shared tools.
- [Creative Code Toronto](https://creativecodetoronto.github.io/) — community meetups focused on creative coding, generative art, and interactive media. Good way to connect with people working at the intersection of code and design.

---

## One Last Thing

Physical computing is a practice. The best way to keep learning is to keep building — pick a sensor you have not tried, connect it, read its values in the Serial Monitor, and figure out what to do with them. The process is always the same one you followed in this course: start simple, test often, and build complexity one layer at a time.

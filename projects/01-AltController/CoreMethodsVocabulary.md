# Core Methods and Vocabulary

### Variables

| Type | Description |
|------|-------------|
| [`int`](https://docs.arduino.cc/language-reference/en/variables/data-types/int/) | Stores whole numbers (integers) from -32,768 to 32,767 |
| [`long`](https://docs.arduino.cc/language-reference/en/variables/data-types/long/) | Stores larger whole numbers—useful for `millis()` values |
| [`bool`](https://docs.arduino.cc/language-reference/en/variables/data-types/bool/) | Stores `true` or `false` values |

### Control Structures

| Structure | Description |
|-----------|-------------|
| [`if`](https://docs.arduino.cc/language-reference/en/structure/control-structure/if/) | Executes code block if a condition is true |
| [`if...else`](https://docs.arduino.cc/language-reference/en/structure/control-structure/else/) | Executes one block if true, another if false |
| [`else if`](https://docs.arduino.cc/language-reference/en/structure/control-structure/else/) | Tests additional conditions after initial `if` |

### Digital I/O

| Method | Description |
|--------|-------------|
| [`pinMode()`](https://docs.arduino.cc/language-reference/en/functions/digital-io/pinmode/) | Configures a pin as INPUT or OUTPUT |
| [`digitalRead()`](https://docs.arduino.cc/language-reference/en/functions/digital-io/digitalread/) | Reads HIGH (1) or LOW (0) from a digital pin |
| [`digitalWrite()`](https://docs.arduino.cc/language-reference/en/functions/digital-io/digitalwrite/) | Writes HIGH or LOW to a digital pin |

### Program Structure

| Method | Description |
|--------|-------------|
| [`setup()`](https://docs.arduino.cc/language-reference/en/structure/sketch/setup/) | Runs once when the program starts—used for initialization |
| [`loop()`](https://docs.arduino.cc/language-reference/en/structure/sketch/loop/) | Runs continuously after setup—main program logic goes here |

### Time Functions

| Method | Description |
|--------|-------------|
| [`delay()`](https://docs.arduino.cc/language-reference/en/functions/time/delay/) | Pauses the program for a specified number of milliseconds |
| [`millis()`](https://docs.arduino.cc/language-reference/en/functions/time/millis/) | Returns the number of milliseconds since the program started |

> **Note:** `delay()` blocks all code execution—the Arduino cannot read inputs or update outputs during the delay. For responsive programs, use `millis()` for non-blocking timing. See [Delay vs Millis Comparison](DelayVsMillis.md) for examples.

### Serial Communication

| Method | Description |
|--------|-------------|
| [`Serial.begin()`](https://docs.arduino.cc/language-reference/en/functions/communication/serial/begin/) | Initializes serial communication at a specified baud rate |
| [`Serial.print()`](https://docs.arduino.cc/language-reference/en/functions/communication/serial/print/) | Prints data to the serial monitor |
| [`Serial.println()`](https://docs.arduino.cc/language-reference/en/functions/communication/serial/println/) | Prints data followed by a new line |

### DFPongController Library

The [DFPongController](https://github.com/DigitalFuturesOCADU/df-pong-controller) library handles Bluetooth communication with the DF Pong game.

#### Setup Methods (call before `begin()`)

| Method | Description |
|--------|-------------|
| `setControllerNumber(int n)` | **Required.** Set your unique controller number (1-242) |
| `setStatusLED(int pin)` | Set LED pin for connection status indicator |
| `begin()` | Initialize BLE connection |

#### Loop Methods (call in `loop()`)

| Method | Description |
|--------|-------------|
| `update()` | **Required.** Call every `loop()` iteration to maintain BLE connection |
| `sendControl(direction)` | Send `UP`, `DOWN`, or `NEUTRAL` to the game |

#### Status Methods

| Method | Returns | Description |
|--------|---------|-------------|
| `isConnected()` | `bool` | True if BLE is connected |
| `isReady()` | `bool` | True if connected AND handshake complete |

#### Direction Constants

| Constant | Value | Description |
|----------|-------|-------------|
| `UP` | 1 | Paddle moves up |
| `DOWN` | 2 | Paddle moves down |
| `NEUTRAL` | 0 | No movement |

#### LED Status Patterns

| Pattern | Meaning |
|---------|---------|
| Slow blink (500ms) | Disconnected, advertising |
| Fast blink (100ms) | Connected, handshaking |
| Solid ON | Ready to play |

### Key Vocabulary

| Term | Definition |
|------|------------|
| **Digital Input** | A signal that is either ON (HIGH/1) or OFF (LOW/0) |
| **GPIO** | General Purpose Input/Output—pins that can be configured as inputs or outputs |
| **Pull-up/Pull-down Resistor** | Resistors that ensure a known state when a switch is open |
| **Debouncing** | Technique to filter out noise from mechanical switch contacts |
| **BLE** | Bluetooth Low Energy—wireless protocol used to connect controller to DF Pong |

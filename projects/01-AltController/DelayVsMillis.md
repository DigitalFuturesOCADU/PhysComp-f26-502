# Delay vs Millis: A Comparison

[← Back to Alt Controller](AltController.md)

## The Problem with `delay()`

When you use `delay()`, the Arduino **stops everything** for the specified time. It cannot:
- Read button presses
- Update LEDs
- Communicate over Serial or Bluetooth
- Do anything else

This is called **blocking** code.

---

## Example: Blinking an LED

### Using `delay()` (Blocking)

```cpp
int ledPin = 13;

void setup() {
  pinMode(ledPin, OUTPUT);
}

void loop() {
  digitalWrite(ledPin, HIGH);   // LED on
  delay(1000);                  // Wait 1 second (BLOCKED!)
  digitalWrite(ledPin, LOW);    // LED off
  delay(1000);                  // Wait 1 second (BLOCKED!)
}
```

**Problem:** If you add a button, it will only be checked twice per cycle—you might miss presses!

---

### Using `millis()` (Non-Blocking)

```cpp
int ledPin = 13;
unsigned long previousMillis = 0;
const long interval = 1000;     // 1 second interval
bool ledState = false;

void setup() {
  pinMode(ledPin, OUTPUT);
}

void loop() {
  unsigned long currentMillis = millis();

  // Check if it's time to toggle the LED
  if (currentMillis - previousMillis >= interval) {
    previousMillis = currentMillis;  // Save the time
    ledState = !ledState;            // Toggle state
    digitalWrite(ledPin, ledState);  // Update LED
  }

  // You can do other things here!
  // Read buttons, update displays, etc.
}
```

**Benefit:** The `loop()` runs continuously. You can check buttons, read sensors, and respond to inputs while still keeping track of time.

---

## When to Use Each

| Use `delay()` when... | Use `millis()` when... |
|-----------------------|------------------------|
| Simple test sketches | Reading inputs (buttons, sensors) |
| Nothing else needs to happen | Multiple things need to happen at once |
| Learning the basics | Running motors, LEDs, and inputs together |
| | Building responsive controllers |

---

## Visual Comparison

### `delay()` Timeline
```
[LED ON]----BLOCKED----[LED OFF]----BLOCKED----[LED ON]...
            1000ms                  1000ms
     (nothing else can happen)
```

### `millis()` Timeline
```
[Check time][Read button][Check time][Update LED][Check time][Read sensor]...
     ↑                        ↑
   "Not yet"               "Time! Toggle LED"
```

---

## Key Pattern: "Blink Without Delay"

This pattern is so common it has a name. The key steps are:

1. **Store** the last time something happened (`previousMillis`)
2. **Check** if enough time has passed (`currentMillis - previousMillis >= interval`)
3. **Update** the stored time when you act (`previousMillis = currentMillis`)
4. **Do the action** (toggle LED, read sensor, etc.)

---

## References

- [Arduino delay() Reference](https://docs.arduino.cc/language-reference/en/functions/time/delay/)
- [Arduino millis() Reference](https://docs.arduino.cc/language-reference/en/functions/time/millis/)
- [Blink Without Delay Tutorial](https://docs.arduino.cc/built-in-examples/digital/BlinkWithoutDelay/)

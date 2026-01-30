# Class 03 | January 23 | Workshop 2

[← Back to Alt Controller](../AltController.md)

## Overview

This class covers a review of Arduino basics from the previous session, introduces general design considerations for custom input switches, and explores second-order interactions. The workshop focuses on connecting custom switches to control a Pong game using Bluetooth.

## References

- [Arduino References](../ArduinoReferences.md)
- [Core Methods & Vocabulary](../CoreMethodsVocabulary.md)
- [DF Pong Game](https://digitalfuturesocadu.github.io/df-pong/game/physComp26-502/)
- [DFPongController Library](https://github.com/DigitalFuturesOCADU/df-pong-controller)

## Review from Last Class

### Arduino Basics

- Arduino allows you to convert data from the physical world into digital data that can be manipulated by software.
- Arduino programs are divided into two main functions: `setup()` and `loop()`
  - `setup()` runs once at the start of the program to initialize variables, pin modes, start using libraries, etc.
  - `loop()` runs continuously after `setup()` has completed, allowing your program to change and respond
- The primary link between Arduino hardware and software is through pins, defined by numbers.
  - Arduino pins can be configured as INPUT or OUTPUT, so you must set this in the `setup()` function using `pinMode()`.
  - We are primarily using the pinMode setting `INPUT_PULLUP` for buttons, which uses an internal resistor to ensure a stable HIGH reading when the button is not pressed.
- Digital pins read two states: HIGH (1) and LOW (0)
  - When using `INPUT_PULLUP`, the button reads HIGH when not pressed, and LOW when pressed.
  - We can read the state of a pin using `digitalRead(pinNumber)`, which returns either HIGH or LOW.
- Conditional statements (`if`, `else`) allow us to interpret data and make decisions in our code.
  - We can use conditional statements to check the state of a button and perform actions based on whether it is pressed or not.
- The Serial Monitor allows us to send and receive text data between the Arduino and our computer.
  - We initialize serial communication in `setup()` using `Serial.begin(baudRate)`
  - We can send data to the Serial Monitor using `Serial.print()` and `Serial.println()`
  - The Serial Monitor is useful for debugging and observing the behavior of our programs.

### Basic Button Example

```cpp
int buttonPin = 2;

void setup() 
{
    pinMode(buttonPin, INPUT_PULLUP);
    pinMode(LED_BUILTIN, OUTPUT);
    Serial.begin(9600);
}

void loop() 
{
    int buttonState = digitalRead(buttonPin);
    
    if (buttonState == LOW) // This is low because we are using INPUT_PULLUP which inverts the reading
    {
        digitalWrite(LED_BUILTIN, HIGH);
        Serial.println("Button pressed");
    } 
    else 
    {
        digitalWrite(LED_BUILTIN, LOW);
        Serial.println("Button not pressed");
    }
}
```

## General Design Considerations

- Any object or material that is conductive can be used to create a custom input switch.
- Consider type/length of wire to create a physical connection between the conductive material and the Arduino pin.
- Consider how the size, shape, weight, and form factor of the conductive material will affect usability.
- These switches can represent notions of:
  - **Location**: Is the user touching a specific area?
  - **Presence**: Is the user touching at all?
  - **Absence**: Is the user not touching?
- All input switches share a common ground pin on the Arduino.
  - Multiple switches can be connected to the same ground pin, but each switch needs its own input pin.
  - Consider how the placement and size of the ground object will affect usability.

## Second Order Interactions with Switches

If statements provide a simple way to read and respond to your switch inputs. However, more complex interactions can be created by tracking the state of the switch over time or comparing multiple switches.

### Second Order Interaction Examples

- **Press Time**: Measure how long a switch is pressed.
- **Click Rate**: Measure how quickly a switch is pressed multiple times.
- **Slot Machine Style**: Randomized outcomes based on switch presses with controlled probabilities.
- **Chorded Inputs**: Combine multiple switches to create new inputs.
- **Sequence Detection**: Recognize specific sequences of switch presses.

---
### Click Example

The key to detecting a "click" (a single press and release) versus just checking if a button is held down is to track the **previous state** of the button. By comparing the current state to the previous state, we can detect the exact moment a button is pressed or released.

```cpp
int buttonPin = 2;

// We need TWO variables to detect changes
int buttonState = HIGH;      // Current state of the button
int lastButtonState = HIGH;  // Previous state of the button (from last loop)

void setup() 
{
    pinMode(buttonPin, INPUT_PULLUP);
    Serial.begin(9600);
    Serial.println("Press the button to see click detection!");
}

void loop() 
{
    // Read the current button state
    buttonState = digitalRead(buttonPin);
    
    // CLICK DETECTED: Button just went from NOT pressed to PRESSED
    // This only happens ONCE at the moment of press
    if (buttonState == LOW && lastButtonState == HIGH) 
    {
        Serial.println("CLICK! Button was just pressed");
    }
    
    // RELEASE DETECTED: Button just went from PRESSED to NOT pressed
    // This only happens ONCE at the moment of release
    if (buttonState == HIGH && lastButtonState == LOW) 
    {
        Serial.println("Button was just released");
    }
    
    // IMPORTANT: Save current state for next time through the loop
    // This becomes the "previous" state on the next iteration
    lastButtonState = buttonState;
}
```

**Key Concept:** Without tracking `lastButtonState`, your code would print "pressed" over and over while the button is held. By comparing current to previous, we detect only the **transition** (the click moment).

---

### Press Time Example

Measure how long a button is held down and report the duration.

```cpp
int buttonPin = 2;

// Variables to track timing
int buttonState = HIGH;
int lastButtonState = HIGH;
unsigned long pressStartTime = 0;

void setup() 
{
    pinMode(buttonPin, INPUT_PULLUP);
    Serial.begin(9600);
    Serial.println("Press and hold the button...");
}

void loop() 
{
    // Read the current button state
    buttonState = digitalRead(buttonPin);
    
    // Button was just pressed (went from HIGH to LOW)
    if (buttonState == LOW && lastButtonState == HIGH) 
    {
        pressStartTime = millis();  // Record when the press started
        Serial.println("Button pressed!");
    }
    
    // Button was just released (went from LOW to HIGH)
    if (buttonState == HIGH && lastButtonState == LOW) 
    {
        unsigned long pressDuration = millis() - pressStartTime;  // Calculate how long it was held
        Serial.print("Button released! Held for ");
        Serial.print(pressDuration);
        Serial.println(" milliseconds");
        
        // You can add different actions based on hold time
        if (pressDuration < 500) 
        {
            Serial.println("-> Short press");
        } 
        else if (pressDuration < 2000) 
        {
            Serial.println("-> Medium press");
        } 
        else 
        {
            Serial.println("-> Long press");
        }
    }
    
    // Remember the current state for next time
    lastButtonState = buttonState;
}
```

---

### Click Rate Example

Count how many times the button is pressed within a time window.

```cpp
int buttonPin = 2;

// Variables to track clicks
int buttonState = HIGH;
int lastButtonState = HIGH;
int clickCount = 0;
unsigned long firstClickTime = 0;
unsigned long clickWindow = 1000;  // Time window in milliseconds (1 second)

void setup() 
{
    pinMode(buttonPin, INPUT_PULLUP);
    Serial.begin(9600);
    Serial.println("Click the button multiple times quickly!");
}

void loop() 
{
    buttonState = digitalRead(buttonPin);
    
    // Button was just pressed
    if (buttonState == LOW && lastButtonState == HIGH) 
    {
        // Is this the first click or a new sequence?
        if (clickCount == 0) 
        {
            firstClickTime = millis();  // Start the timer
        }
        
        clickCount = clickCount + 1;  // Add to our count
        Serial.print("Click #");
        Serial.println(clickCount);
    }
    
    // Check if the time window has passed
    if (clickCount > 0 && millis() - firstClickTime > clickWindow) 
    {
        Serial.print("You clicked ");
        Serial.print(clickCount);
        Serial.println(" times in 1 second!");
        
        // Different actions based on click count
        if (clickCount == 1) 
        {
            Serial.println("-> Single click");
        } 
        else if (clickCount == 2) 
        {
            Serial.println("-> Double click!");
        } 
        else if (clickCount >= 3) 
        {
            Serial.println("-> Triple click or more!");
        }
        
        Serial.println("---");
        clickCount = 0;  // Reset for next sequence
    }
    
    lastButtonState = buttonState;
}
```

---

### Slot Machine Style Example

Each button press gives a random outcome with controlled probabilities.

```cpp
int buttonPin = 2;

int buttonState = HIGH;
int lastButtonState = HIGH;

void setup() 
{
    pinMode(buttonPin, INPUT_PULLUP);
    Serial.begin(9600);
    
    // Use an unconnected analog pin to seed random numbers
    randomSeed(analogRead(A0));
    
    Serial.println("Press the button to spin!");
}

void loop() 
{
    buttonState = digitalRead(buttonPin);
    
    // Button was just pressed
    if (buttonState == LOW && lastButtonState == HIGH) 
    {
        Serial.println("Spinning...");
        
        // Generate a random number from 1 to 100
        int randomNumber = random(1, 101);
        
        Serial.print("Random roll: ");
        Serial.println(randomNumber);
        
        // Different outcomes based on probability
        // 50% chance (numbers 1-50)
        if (randomNumber <= 50) 
        {
            Serial.println("Result: NOTHING - Try again!");
        }
        // 30% chance (numbers 51-80)
        else if (randomNumber <= 80) 
        {
            Serial.println("Result: SMALL WIN!");
        }
        // 15% chance (numbers 81-95)
        else if (randomNumber <= 95) 
        {
            Serial.println("Result: BIG WIN!!");
        }
        // 5% chance (numbers 96-100)
        else 
        {
            Serial.println("Result: JACKPOT!!!");
        }
        
        Serial.println("---");
    }
    
    lastButtonState = buttonState;
}
```

---

### Chorded Inputs Example

Combine multiple buttons pressed together to create new actions.

```cpp
int buttonPin1 = 2;
int buttonPin2 = 3;
int buttonPin3 = 4;

int button1State = HIGH;
int button2State = HIGH;
int button3State = HIGH;
int lastButton1State = HIGH;
int lastButton2State = HIGH;
int lastButton3State = HIGH;

void setup() 
{
    pinMode(buttonPin1, INPUT_PULLUP);
    pinMode(buttonPin2, INPUT_PULLUP);
    pinMode(buttonPin3, INPUT_PULLUP);
    Serial.begin(9600);
    Serial.println("Try pressing buttons individually or in combinations!");
}

void loop() 
{
    // Read all three buttons
    button1State = digitalRead(buttonPin1);
    button2State = digitalRead(buttonPin2);
    button3State = digitalRead(buttonPin3);
    
    // Check if anything changed
    int somethingChanged = (button1State != lastButton1State) || 
                           (button2State != lastButton2State) || 
                           (button3State != lastButton3State);
    
    if (somethingChanged) 
    {
        // Count how many buttons are pressed
        int pressedCount = 0;
        if (button1State == LOW) pressedCount = pressedCount + 1;
        if (button2State == LOW) pressedCount = pressedCount + 1;
        if (button3State == LOW) pressedCount = pressedCount + 1;
        
        // ALL THREE buttons pressed = Super Chord
        if (button1State == LOW && button2State == LOW && button3State == LOW) 
        {
            Serial.println("SUPER CHORD: All 3 buttons! -> Ultimate action!");
        }
        // Two-button combinations
        else if (button1State == LOW && button2State == LOW && button3State == HIGH) 
        {
            Serial.println("CHORD: Buttons 1+2 -> Action AB");
        }
        else if (button1State == LOW && button2State == HIGH && button3State == LOW) 
        {
            Serial.println("CHORD: Buttons 1+3 -> Action AC");
        }
        else if (button1State == HIGH && button2State == LOW && button3State == LOW) 
        {
            Serial.println("CHORD: Buttons 2+3 -> Action BC");
        }
        // Single button presses
        else if (button1State == LOW && button2State == HIGH && button3State == HIGH) 
        {
            Serial.println("Button 1 only -> Action A");
        }
        else if (button1State == HIGH && button2State == LOW && button3State == HIGH) 
        {
            Serial.println("Button 2 only -> Action B");
        }
        else if (button1State == HIGH && button2State == HIGH && button3State == LOW) 
        {
            Serial.println("Button 3 only -> Action C");
        }
        // All released
        else 
        {
            Serial.println("All buttons released");
        }
    }
    
    // Remember states for next time
    lastButton1State = button1State;
    lastButton2State = button2State;
    lastButton3State = button3State;
}
```

---

### Sequence Detection Example

Recognize a specific pattern of button presses based on physical location. By arranging buttons in a row (Left, Center, Right), you can detect directional gestures like swipes!

```cpp
// Buttons arranged physically from left to right
int leftButtonPin = 2;    // Leftmost button
int centerButtonPin = 3;  // Center button
int rightButtonPin = 4;   // Rightmost button

int leftState = HIGH;
int centerState = HIGH;
int rightState = HIGH;
int lastLeftState = HIGH;
int lastCenterState = HIGH;
int lastRightState = HIGH;

// Store the last 3 button presses to detect patterns
// 1 = Left, 2 = Center, 3 = Right
int inputHistory[] = {0, 0, 0};
int inputIndex = 0;

void setup() 
{
    pinMode(leftButtonPin, INPUT_PULLUP);
    pinMode(centerButtonPin, INPUT_PULLUP);
    pinMode(rightButtonPin, INPUT_PULLUP);
    Serial.begin(9600);
    Serial.println("Arrange 3 buttons in a row: [LEFT] [CENTER] [RIGHT]");
    Serial.println("Touch them in sequence to create directional gestures!");
    Serial.println("Try: Left->Center->Right for a RIGHT SWIPE");
    Serial.println("Try: Right->Center->Left for a LEFT SWIPE");
    Serial.println("---");
}

void loop() 
{
    // Read all three buttons
    leftState = digitalRead(leftButtonPin);
    centerState = digitalRead(centerButtonPin);
    rightState = digitalRead(rightButtonPin);
    
    // Check for new button presses and record them
    int newPress = 0;  // Track if a new press happened
    
    // Left button pressed
    if (leftState == LOW && lastLeftState == HIGH) 
    {
        Serial.println("LEFT pressed");
        inputHistory[inputIndex] = 1;
        inputIndex = inputIndex + 1;
        newPress = 1;
    }
    
    // Center button pressed
    if (centerState == LOW && lastCenterState == HIGH) 
    {
        Serial.println("CENTER pressed");
        inputHistory[inputIndex] = 2;
        inputIndex = inputIndex + 1;
        newPress = 1;
    }
    
    // Right button pressed
    if (rightState == LOW && lastRightState == HIGH) 
    {
        Serial.println("RIGHT pressed");
        inputHistory[inputIndex] = 3;
        inputIndex = inputIndex + 1;
        newPress = 1;
    }
    
    // Once we have 3 inputs, check for patterns
    if (inputIndex >= 3) 
    {
        // Check for RIGHT SWIPE: Left(1) -> Center(2) -> Right(3)
        if (inputHistory[0] == 1 && inputHistory[1] == 2 && inputHistory[2] == 3) 
        {
            Serial.println(">>> RIGHT SWIPE detected! >>>");
        }
        // Check for LEFT SWIPE: Right(3) -> Center(2) -> Left(1)
        else if (inputHistory[0] == 3 && inputHistory[1] == 2 && inputHistory[2] == 1) 
        {
            Serial.println("<<< LEFT SWIPE detected! <<<");
        }
        // Check for OUTWARD gesture: Center(2) -> Left(1) -> Right(3) OR Center(2) -> Right(3) -> Left(1)
        else if (inputHistory[0] == 2 && inputHistory[1] == 1 && inputHistory[2] == 3) 
        {
            Serial.println("<-> EXPAND gesture detected! <->");
        }
        else if (inputHistory[0] == 2 && inputHistory[1] == 3 && inputHistory[2] == 1) 
        {
            Serial.println("<-> EXPAND gesture detected! <->");
        }
        // Check for INWARD gesture: Left(1) -> Right(3) -> Center(2) OR Right(3) -> Left(1) -> Center(2)
        else if (inputHistory[0] == 1 && inputHistory[1] == 3 && inputHistory[2] == 2) 
        {
            Serial.println(">-< PINCH gesture detected! >-<");
        }
        else if (inputHistory[0] == 3 && inputHistory[1] == 1 && inputHistory[2] == 2) 
        {
            Serial.println(">-< PINCH gesture detected! >-<");
        }
        // Check for DOUBLE TAP patterns (same button twice then different)
        else if (inputHistory[0] == inputHistory[1]) 
        {
            Serial.print("Double tap on ");
            if (inputHistory[0] == 1) Serial.print("LEFT");
            else if (inputHistory[0] == 2) Serial.print("CENTER");
            else Serial.print("RIGHT");
            Serial.println(", then moved.");
        }
        else 
        {
            Serial.println("Pattern not recognized. Try a swipe!");
        }
        
        // Reset for next sequence
        inputIndex = 0;
        Serial.println("---");
    }
    
    // Remember states for next time
    lastLeftState = leftState;
    lastCenterState = centerState;
    lastRightState = rightState;
}
```
## DFPongController Library

The [DFPongController](https://github.com/DigitalFuturesOCADU/df-pong-controller) is a beginner-friendly Arduino library for creating Bluetooth Low Energy (BLE) controllers that connect to the DF Pong browser game.

### Setup Checklist

#### Step 1: Verify You Have a Supported Browser

The DF Pong game uses the [Web Bluetooth API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Bluetooth_API), which has limited browser support.

| Browser | Desktop | Mobile |
|---------|---------|--------|
| Chrome | ✅ Supported | ✅ Android only |
| Edge | ✅ Supported | ✅ Android only |
| Opera | ✅ Supported | ✅ Android only |
| Safari | ❌ Not supported | ❌ Not supported |
| Firefox | ❌ Not supported | ❌ Not supported |
| iOS (all browsers) | ❌ Not supported | ❌ Not supported |

**Important Notes:**
- **iOS devices** (iPhone/iPad) do not support Web Bluetooth in any browser, including Chrome
- **Use Chrome on desktop** for the most reliable experience
- On Android, use Chrome, Edge, or Opera

#### Step 2: Install the Library
- [ ] Open Arduino IDE
- [ ] Go to **Sketch → Include Library → Manage Libraries...**
- [ ] Search for **"DFPongController"**
- [ ] Click **Install With Dependencies**

#### Step 3: Load the Example Sketch
- [ ] Go to **File → Examples → DFPongController → 01_SimpleDigital**

#### Step 4: Configure Your Controller Number
- [ ] Find the line: `int controllerNumber = 1;`
- [ ] Change `1` to your assigned number (see Controller Number Assignments table)

#### Step 5: Check Pin Wiring
- [ ] Default pins are **2** (UP) and **3** (DOWN)
- [ ] Adjust code OR wiring to match your setup

#### Step 6: Upload the Code
- [ ] Select your board: **Tools → Board → Arduino UNO R4 WiFi**
- [ ] Select your port: **Tools → Port → (your Arduino)**
- [ ] Click **Upload** (→ arrow button)

#### Step 7: Verify LED Status

| LED Pattern | Meaning |
|-------------|---------|
| Slow blink (500ms) | Disconnected, advertising |
| Fast blink (100ms) | Connected, handshaking |
| Solid ON | Ready to play! |

#### Step 8: Connect to the Game
- [ ] Open the game: https://digitalfuturesocadu.github.io/df-pong/game/physComp26-502/
- [ ] Select your controller number from the dropdown
- [ ] Click **Connect**
- [ ] Select your device in the Bluetooth popup
- [ ] Confirm LED goes solid ON

### Quick Start Code

```cpp
#include <DFPongController.h>

int upButtonPin = 2;
int downButtonPin = 3;

int controllerNumber = 1;  // <-- CHANGE THIS TO YOUR ASSIGNED NUMBER!

DFPongController controller;

void setup() 
{
    pinMode(upButtonPin, INPUT_PULLUP);
    pinMode(downButtonPin, INPUT_PULLUP);
    
    controller.setControllerNumber(controllerNumber);
    controller.setStatusLED(LED_BUILTIN);
    controller.begin();
}

void loop() 
{
    controller.update();  // Required every loop!
    
    // Read the current button states
    int upButtonState = digitalRead(upButtonPin);
    int downButtonState = digitalRead(downButtonPin);
    
    // Check which button is pressed and send the appropriate control
    if (upButtonState == LOW) 
    {
        controller.sendControl(UP);
    } 
    else if (downButtonState == LOW) 
    {
        controller.sendControl(DOWN);
    } 
    else 
    {
        controller.sendControl(NEUTRAL);
    }
}
```

### API Reference

#### Setup Methods

| Method | Description |
|--------|-------------|
| `setControllerNumber(int n)` | **Required.** Set your unique number (1-242) |
| `setStatusLED(int pin)` | Set LED pin for connection status |
| `setDebug(bool enabled)` | Enable Serial debug messages |
| `begin()` | Initialize BLE with default name |

#### Loop Methods

| Method | Description |
|--------|-------------|
| `update()` | **Required.** Call every `loop()` iteration |
| `sendControl(int direction)` | Send `UP`, `DOWN`, or `NEUTRAL` |

#### Status Methods

| Method | Returns | Description |
|--------|---------|-------------|
| `isConnected()` | bool | True if BLE connected |
| `isReady()` | bool | True if connected AND handshake complete |
| `getRSSI()` | int | Signal strength in dBm |

#### Direction Constants

| Constant | Value | Description |
|----------|-------|-------------|
| `UP` | 1 | Paddle moves up |
| `DOWN` | 2 | Paddle moves down |
| `NEUTRAL` | 0 | No movement |



## Workshop Activities

Get your first switches controlling the Pong Game: https://digitalfuturesocadu.github.io/df-pong/game/physComp26-502/ and Play Against Someone.

### Installing the DFPongController Library

1. Open the Library Manager (the books icon in the left toolbar)
2. Search for "DFPongController"
3. Click "Install"

### Loading the Example Sketch

1. Open File > Examples > DFPongController > 01_SimpleDigital
2. Adjust either your wiring or the pin numbers in the code to match your setup
   - The example code uses pins 2 and 3 for the two directions
3. Change your controller number to match your assigned number (see table below)
4. Upload the code to your Arduino

### Controller Number Assignments

| Device # | Name          |     | Device # | Name     |
|:--------:|:--------------|:---:|:--------:|:---------|
| 1        | Nick          |  ▪  | 20       | Emma     |
| 2        | Marie-Josepha |  ▪  | 21       | Nia      |
| 3        | Pj            |  ▪  | 22       | Saif     |
| 4        | Lisa          |  ▪  | 23       | Malini   |
| 5        | Maddox        |  ▪  | 24       | Emily    |
| 6        | Jena          |  ▪  | 25       | Alicia   |
| 7        | Casimir       |  ▪  | 26       | Brooklyn |
| 8        | Jackson       |  ▪  | 27       | Quinn    |
| 9        | Nico          |  ▪  | 28       | Vedanti  |
| 10       | Estelle       |  ▪  | 29       | Shaine   |
| 11       | Cathy         |  ▪  | 30       | Natalie  |
| 12       | Ashna         |  ▪  | 31       | Cenker   |
| 13       | Rea           |  ▪  | 32       | Aubrie   |
| 14       | Chante        |  ▪  | 33       | Saarukan |
| 15       | Vivian        |  ▪  | 34       | Hayah    |
| 16       | Nayha         |  ▪  | 35       | Elaine   |
| 17       | Jhonna        |  ▪  | 36       | Rhaven   |
| 18       | Aisha         |  ▪  | 37       | Cedric   |
| 19       | Tiffany       |     |          |          |

### Connecting to the Pong Game

1. Once the code is uploaded, the built-in LED will flash to show that it is ready to connect
2. Open the Pong Game in your web browser: https://digitalfuturesocadu.github.io/df-pong/game/physComp26-502/
3. Select your controller from the dropdown menu for either Player 1 or Player 2
4. Press Connect
5. This will trigger the Bluetooth Connection window in the browser
   - There should only be one device listed in this window. If there is more than one, you forgot to set your unique controller number in the code—go back and fix it.
6. Select the Device and press Connect
7. Once connected, you should see the built-in LED on your Arduino turn solid ON
8. You will also see your name on screen as the connected player
9. Press Start and test your controller!
10. Repeat for Player 2

## Resources

- [DFPongController Library](https://github.com/DigitalFuturesOCADU/DFPongController)
- [Pong Game](https://digitalfuturesocadu.github.io/df-pong/game/physComp26-502/)

## Deliverable

Workshop 2 submission due at end of class.

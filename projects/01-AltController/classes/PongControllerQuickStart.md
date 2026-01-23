# Pong Controller Quick Start

[← Back to Class 03](class03-Jan23.md)

```cpp
#include <DFPongController.h>

int upButtonPin = 2;
int downButtonPin = 3;

DFPongController controller;

void setup() 
{
    pinMode(upButtonPin, INPUT_PULLUP);
    pinMode(downButtonPin, INPUT_PULLUP);
    
    // IMPORTANT: Set YOUR unique controller number (1-242)
    controller.setControllerNumber(1);  // <-- CHANGE THIS!
    
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

## Game Link

https://digitalfuturesocadu.github.io/df-pong/game/physComp26-502/

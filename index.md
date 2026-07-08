# Automated Smart Bridge
The Automated Smart Bridge isn’t any normal bridge; it raises and lowers itself based off environmental conditions. When its sensors detects moisture, it displays a flood alert while simultaneously raising the bridge. The purpose of this bridge is to ensure that people can cross it safely during floody conditions.
```HTML 
<!--- This is an HTML comment in Markdown -->
<!--- Anything between these symbols will not render on the published site -->
```

| **Engineer** | **School** | **Area of Interest** | **Grade** |
|:--:|:--:|:--:|:--:|
| Logan W | Irvine HS | Structural Engineering | Incoming Junior

**Replace the BlueStamp logo below with an image of yourself and your completed project. Follow the guide [here](https://tomcam.github.io/least-github-pages/adding-images-github-pages-site.html) if you need help.**

![Headstone Image](logo.svg)
  
# Final Milestone

**Don't forget to replace the text below with the embedding for your milestone video. Go to Youtube, click Share -> Embed, and copy and paste the code to replace what's below.**

<iframe width="560" height="315" src="https://www.youtube.com/embed/F7M7imOVGug" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

For your final milestone, explain the outcome of your project. Key details to include are:
- What you've accomplished since your previous milestone
- What your biggest challenges and triumphs were at BSE
- A summary of key topics you learned about
- What you hope to learn in the future after everything you've learned at BSE



# Second Milestone

**Don't forget to replace the text below with the embedding for your milestone video. Go to Youtube, click Share -> Embed, and copy and paste the code to replace what's below.**

<iframe width="560" height="315" src="https://www.youtube.com/embed/y3VAmNlER5Y" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

For your second milestone, explain what you've worked on since your previous milestone. You can highlight:
- Technical details of what you've accomplished and how they contribute to the final goal
- What has been surprising about the project so far
- Previous challenges you faced that you overcame
- What needs to be completed before your final milestone 

# First Milestone

<iframe width="560" height="315" src="https://www.youtube.com/embed/bIdyJx_JLe0?si=ucU84TKvC5E5MqnM" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

Different Components
- Soil Moisture Sensor: Detects moisture and sends signals to the Arduino
- Arduino: Acts as a motherboard and recieves signals from the Soil Moisture Sensor and sends signals to output components
- Servo: A motor that recieves signals from the Arduino and raises and lowers the bridge with its arm.
- OLED: A screen that recieves signals from the Ardunio and displays text
- Red LED: A light that recieves signals from the Arduino and lights up when a flood is detected
  
Challenges I Overcame
- The physical wiring of the Arduino was the hardest part since I had to learn the basics of wiring it from scratch, now that I have finished the base project, I feel more confident about my wiring abilities
- It took trial and error, but the effort to connect the wires to the right places was incredibly rewarding once the bridge started working
  
Future Iterations
- I plan to make modifications to the bridge to improve safety


# Schematics 
Here's where you'll put images of your schematics. [Tinkercad](https://www.tinkercad.com/blog/official-guide-to-tinkercad-circuits) and [Fritzing](https://fritzing.org/learning/) are both great resoruces to create professional schematic diagrams, though BSE recommends Tinkercad becuase it can be done easily and for free in the browser. 

# Code
Here's where you'll put your code. The syntax below places it into a block of code. Follow the guide [here]([url](https://www.markdownguide.org/extended-syntax/)) to learn how to customize it to your project needs. 

```c++
#include <Servo.h>           // Include Servo library
#include <Wire.h>            // Include I2C library
#include <Adafruit_GFX.h>    // Include Adafruit GFX library
#include <Adafruit_SSD1306.h> // Include Adafruit SSD1306 OLED library

// OLED display configuration
#define SCREEN_WIDTH 128
#define SCREEN_HEIGHT 64
#define OLED_RESET -1
Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, OLED_RESET);

Servo tap_servo;

// Pin definitions
const int estopPin = 2;       // NEW: Your Gikfun E-Stop button (Must be Pin 2 for interrupts)
const int sensor_pin = 5;    // Digital output (DO) pin of the soil moisture sensor
const int tap_servo_pin = 4; // Servo signal pin
const int sensor_vcc = 7;    // Power pin for the sensor
const int led_pin = 6;       // LED to indicate flood alert/E-stop blinking

int val;

// Non-blocking timer variables (Replaces delay(500))
unsigned long previousMillis = 0;
const long interval = 500; 

// E-Stop control variables
volatile bool estopTriggered = false;
unsigned long blinkPreviousMillis = 0;
bool ledState = LOW;

void setup() {
  Serial.begin(9600);
  
  pinMode(sensor_pin, INPUT);      // Soil sensor digital output
  pinMode(sensor_vcc, OUTPUT);    // Sensor power pin
  pinMode(led_pin, OUTPUT);       // LED pin
  
  // NEW: Initialize E-Stop pin with internal pullup
  pinMode(estopPin, INPUT_PULLUP);
  
  digitalWrite(sensor_vcc, HIGH); // Turn on the sensor
  tap_servo.attach(tap_servo_pin);

  // Initialize the OLED
  if (!display.begin(SSD1306_SWITCHCAPVCC, 0x3C)) { // I2C address 0x3C
    Serial.println("SSD1306 allocation failed");
    for (;;); // Don't proceed, loop forever
  }

  // Display startup message
  display.clearDisplay();
  display.setTextSize(2);
  display.setTextColor(WHITE);
  display.setCursor(10, 25); 
  display.println("Safe Level"); 
  display.display();
  delay(2000); // Kept here just for startup splash screen visual pause
  display.clearDisplay(); 
  
  // NEW: Attach the physical interrupt to Pin 2
  attachInterrupt(digitalPinToInterrupt(estopPin), emergencyStop, FALLING);
}

void loop() {
  // 1. CRITICAL SAFETY LOCKDOWN MECHANISM
  if (estopTriggered) {
    tap_servo.detach(); // Instantly cut power to the servo so the bridge drops/freewheels
    
    // Update screen once to alert operators
    display.clearDisplay();
    display.setTextSize(2);
    display.setTextColor(WHITE);
    display.setCursor(5, 25);
    display.println("E-STOP LOCK");
    display.display();
    
    // Enter infinite lockdown loop: Rapidly flashes your alert LED
    while (true) {
      unsigned long currentMillis = millis();
      if (currentMillis - blinkPreviousMillis >= 100) { // Flashes every 100ms
        blinkPreviousMillis = currentMillis;
        ledState = !ledState;
        digitalWrite(led_pin, ledState);
      }
    }
  }

  // 2. NORMAL RUNTIME SEQUENCE (Runs every 500ms without blocking the processor)
  unsigned long currentMillis = millis();
  if (currentMillis - previousMillis >= interval) {
    previousMillis = currentMillis;
    
    val = digitalRead(sensor_pin); // Read the water sensor

    if (val == LOW) { // LOW means water detected
      tap_servo.write(90);         // Rotate servo to 90°
      digitalWrite(led_pin, HIGH); // Turn on LED
      displayStatus("Flood Alert");// Display flood alert message
    } else { // HIGH means no water detected
      tap_servo.write(0);          // Rotate servo back to 0°
      digitalWrite(led_pin, LOW);  // Turn off LED
      displayStatus("Safe Level"); // Display safe message
    }
  }
}

// Function to display status on OLED
void displayStatus(const char* message) {
  display.clearDisplay();          
  display.setTextSize(2);          
  display.setTextColor(WHITE);     
  display.setCursor(10, 25);       
  display.println(message);        
  display.display();               
}

// NEW: Interrupt Service Routine for instant button tracking
void emergencyStop() {
  static unsigned long lastInterruptTime = 0;
  unsigned long interruptTime = millis();
  
  // Software debouncing to ignore hardware vibration noise
  if (interruptTime - lastInterruptTime > 200) {
    estopTriggered = true;
  }
  lastInterruptTime = interruptTime;
}
```

# Bill of Materials
Here's where you'll list the parts in your project. To add more rows, just copy and paste the example rows below.
Don't forget to place the link of where to buy each component inside the quotation marks in the corresponding row after href =. Follow the guide [here]([url](https://www.markdownguide.org/extended-syntax/)) to learn how to customize this to your project needs. 

| **Part** | **Note** | **Price** | **Link** |
|:--:|:--:|:--:|:--:|
| Arduino Uno | What the item is used for | $Price | <a href="https://shorturl.at/KSbxx/"> Link </a> |
| OLED Module | What the item is used for | $Price | <a href="https://www.amazon.com/Arduino-A000066-ARDUINO-UNO-R3/dp/B008GRTSV6/"> Link </a> |
| Breadboard | What the item is used for | $Price | <a href="https://www.amazon.com/Arduino-A000066-ARDUINO-UNO-R3/dp/B008GRTSV6/"> Link </a> |
| Micro Servo | What the item is used for | $Price | <a href="https://www.amazon.com/Arduino-A000066-ARDUINO-UNO-R3/dp/B008GRTSV6/"> Link </a> |
| Female Jumper Wires | What the item is used for | $Price | <a href="https://www.amazon.com/Arduino-A000066-ARDUINO-UNO-R3/dp/B008GRTSV6/"> Link </a> |
| Male Jumper Wires | What the item is used for | $Price | <a href="https://www.amazon.com/Arduino-A000066-ARDUINO-UNO-R3/dp/B008GRTSV6/"> Link </a> |
| Soil Moisture Sensor | What the item is used for | $Price | <a href="https://www.amazon.com/Arduino-A000066-ARDUINO-UNO-R3/dp/B008GRTSV6/"> Link </a> |
| Red Button | What the item is used for | $Price | <a href="https://www.amazon.com/Arduino-A000066-ARDUINO-UNO-R3/dp/B008GRTSV6/"> Link </a> |
| Red LED | What the item is used for | $Price | <a href="https://www.amazon.com/Arduino-A000066-ARDUINO-UNO-R3/dp/B008GRTSV6/"> Link </a> |
| Cardboard | What the item is used for | $Price | <a href="https://www.amazon.com/Arduino-A000066-ARDUINO-UNO-R3/dp/B008GRTSV6/"> Link </a> |
| Construction Paper | What the item is used for | $Price | <a href="https://www.amazon.com/Arduino-A000066-ARDUINO-UNO-R3/dp/B008GRTSV6/"> Link </a> |
| Battery Pack | What the item is used for | $Price | <a href="https://www.amazon.com/Arduino-A000066-ARDUINO-UNO-R3/dp/B008GRTSV6/"> Link </a> |


# Other Resources/Examples
One of the best parts about Github is that you can view how other people set up their own work. Here are some past BSE portfolios that are awesome examples. You can view how they set up their portfolio, and you can view their index.md files to understand how they implemented different portfolio components.
- [Example 1](https://trashytuber.github.io/YimingJiaBlueStamp/)
- [Example 2](https://sviatil0.github.io/Sviatoslav_BSE/)
- [Example 3](https://arneshkumar.github.io/arneshbluestamp/)

To watch the BSE tutorial on how to create a portfolio, click here.

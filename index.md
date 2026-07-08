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
| Arduino Uno | What the item is used for | $Price | <a href="https://www.amazon.com/ELEGOO-Board-ATmega328P-ATMEGA16U2-Compliant/dp/B01EWOE0UU/ref=sr_1_2_sspa?dib=eyJ2IjoiMSJ9.f3j_Bgk5w5uiGML1DWkfl0esThnO81D4PzyHpk4VEejBGIq5HPEOCHJVyJ8VKvAZyAU3h9NlB_EOtGuanbt4cUz0bSo2Gu1lb9ian4pFVYC6LvBJ80VICkSzPfo2Lgg2fyn2PLLie0BF1ewB3_H7UqTbLZggxMaD47wMRHb40tvgxThtgf5tgNTm_uSklByC5KAOroWLvL6AHvnfcW6_-zuRdgWsU4J63RfAGXiw6Tra8DCE7hey7DmL1rJHw38_eCN5_dgITyZLBwShFUgpGg0lIme2dTNcQt1jhG3hgcE.Q_aKIgO19m6uVb-aoZkbvVjJlnQuC_-lTFjfu8EREWg&dib_tag=se&keywords=arduino+uno&qid=1783537023&s=industrial&sr=1-2-spons&sp_csd=d2lkZ2V0TmFtZT1zcF9hdGY&psc=1"> Link </a> |
| OLED Module | What the item is used for | $Price | <a href="https://www.amazon.com/ELEGOO-Display-Compact-Self-Luminous-Projects/dp/B0D2RMQQHR/ref=sr_1_1_sspacrid=304YHUORZ27MD&dib=eyJ2IjoiMSJ9.oAZKdk9yLjCgM4o7DR9IOk7GEFPkm_nvsBW48PaUzBOpG4swxSpAAclrO102VNiVTlz9gaad0in55iUnNXdHbNmEAy9oNBKIm4xp9vNDoEcyHVgAM18CHOEURBOAAeToDiIDmlVbyYrEJgpOQI-YDFuZdXq30sLe7W8Fxm0BFzP7BwORKzBGZzGD56YtNRC0ApS-IbfGdypIo5VImYTZz3HjzS0Yndvfs7aNPBeKQo0XJiMswTEftbZbfTWz8gU3WIpABlXPn6iQXJpPexiAogsXEsiaOsTHuNKodN50U.0_b47w0BVblBmYQ7S_jrR7ngTI98cLTLfzzCfqKFOk4&dib_tag=se&keywords=oled%2Bmodule&qid=1783537269&s=industial&sprefix=oled%2Bmodu%2Cindustrial%2C186&sr=1-1spons&sp_csd=d2lkZ2V0TmFtZT1zcF9hdGY&th=1"> Link </a> |
| Breadboard | What the item is used for | $Price | <a href="https://www.amazon.com/Breadboards-Solderless-Breadboard-Distribution-Connecting/dp/B07DL13RZH/ref=sr_1_1_sspa?crid=2NP9TVEI9YJ4A&dib=eyJ2IjoiMSJ9.5Z5yTwL-oa1r18Ah_zf9OdziZtM7NtJzJKlf3z7Il1SZx-ph1l3ITB17WTZqmuHLE8rcxl4AJ3kJyvvqSFnxIw_mmmpakr_40tjpQaGTleMctk9x3ocMPnKFtmgK-heQ6kwxLurzKtJ0BVYli881oIE7HAVodJBg1jf7PQARsJucej13gxhwUeBwfHYtmZAEOkHC8t6B840zHSbLbgtBayfAElMkRCtBlJWlX0MaHD1F8PU51oWy18nfZ7PvnhNEyPKb8wYdDGHahZrcCmE7fyoPofpgdA5V0q1r_vtx21A.eieMym2UsR4nDnU1Tlh8hgEi-K1ZvfYuDblmQTrsTFo&dib_tag=se&keywords=breadboard&qid=1783537382&s=industrial&sprefix=breadboar%2Cindustrial%2C203&sr=1-1-spons&sp_csd=d2lkZ2V0TmFtZT1zcF9hdGY&psc=1"> Link </a> |
| Micro Servo | What the item is used for | $Price | <a href="https://www.amazon.com/Servo-Servos-Helicopter-Airplane-Controls/dp/B0BJQ2QTHG/ref=sr_1_2_sspa?crid=5R4E6BPGGMGU&dib=eyJ2IjoiMSJ9.hpJG43Djr80Gjf-qpoo4h3_Bi-FcWHePXFoDo2a-iXlVhNWiYKO1XKWC9l0vZnSxUUsrj9FWMJeSND15Lh6sU_kWvOdWtcs_Eu313H3U_hPxl60GsbkVqacypHxKKvf-5Hug1hXBgPfE4fS1w5rqO8ZOUCz_2aAUTrTo0OZjqgD8jngG3re0oRIbFlhnMd6DLuxwMKcMrXDS3RNc1YB7-Dhq6mYGvuOI8mGqDx_EihDxR6uCy9bYiY7LILiiHfthL5R3v5VlAWF84fABHazlecP3sW9xWDnJTxVu8yujB5o.7pZH0bGyvUiyepolOOf1KzKJ4Wixm-poxHZCtpPTt3s&dib_tag=se&keywords=micro%2Bservo&qid=1783537465&s=industrial&sprefix=micro%2Bserv%2Cindustrial%2C200&sr=1-2-spons&sp_csd=d2lkZ2V0TmFtZT1zcF9hdGY&th=1"> Link </a> |
| Female Jumper Wires | What the item is used for | $Price | <a href="https://www.amazon.com/California-JOS-Breadboard-Optional-Multicolored/dp/B0BRTJQGS6/ref=sr_1_3?crid=16LB2LRUIZU6H&dib=eyJ2IjoiMSJ9.BMvrgU_YjBIEPI70oBIcSWa3MkascqwcqGWz0yYndqacFBGJ7cfkxAuL3_nZXgYGa7nwmce8zIXo-U22N1UUaPGgNUx4T9UlCG-9_w1-P7na82losEQNgnm_eVapEifNKerimt8_QSpxABrxfmTAeUg-TN2bJRRiQlaaZbsr6Gcw-_RU7IWqXoxPpWbE4ahjK666OPJv8s_-LuNlToNpISJWUkiToG1NWe_SMrQ0PFgdvYMWhzirtFmOm-JP8HWbJPythDrZANLir3bP3CGDPT6uPlP1VpL1-TkOX_2E8d4.sggP3P7wXTfRuwIrh_hn0ynGH7fVGm91oVe1orJ-1co&dib_tag=se&keywords=male%2Bjumper%2Bwires&qid=1783537528&s=industrial&sprefix=fmale%2Bjumper%2Bwires%2Cindustrial%2C177&sr=1-3&th=1"> Link </a> |
| Male Jumper Wires | What the item is used for | $Price | <a href="https://www.amazon.com/California-JOS-Breadboard-Optional-Multicolored/dp/B0BRTJQZRD/ref=sr_1_3?crid=16LB2LRUIZU6H&dib=eyJ2IjoiMSJ9.BMvrgU_YjBIEPI70oBIcSWa3MkascqwcqGWz0yYndqacFBGJ7cfkxAuL3_nZXgYGa7nwmce8zIXo-U22N1UUaPGgNUx4T9UlCG-9_w1-P7na82losEQNgnm_eVapEifNKerimt8_QSpxABrxfmTAeUg-TN2bJRRiQlaaZbsr6Gcw-_RU7IWqXoxPpWbE4ahjK666OPJv8s_-LuNlToNpISJWUkiToG1NWe_SMrQ0PFgdvYMWhzirtFmOm-JP8HWbJPythDrZANLir3bP3CGDPT6uPlP1VpL1-TkOX_2E8d4.sggP3P7wXTfRuwIrh_hn0ynGH7fVGm91oVe1orJ-1co&dib_tag=se&keywords=male%2Bjumper%2Bwires&qid=1783537528&s=industrial&sprefix=fmale%2Bjumper%2Bwires%2Cindustrial%2C177&sr=1-3&th=1"> Link </a> |
| Soil Moisture Sensor | What the item is used for | $Price | <a href="https://www.amazon.com/MTDELE-10Pcs-Moisture-Sensor-3-3-5V/dp/B0F27WB4RK/ref=sr_1_3_sspa?crid=DZHGJCMBS7HF&dib=eyJ2IjoiMSJ9.3s60fRMlbT6tO9vQNE6Vq_7kYwZrdtP49TjwOK0S8BqeTqhP6oQ2UomhnGJL1UC22LsoNoc-O0S0aexpfMJEgo2Jnnyy41gozfQMYi4NcaQS8eoUrhYMlCTEMJ3Yo3izmsUaz0sWV5BvZ84TfB4QYPVYAhCjj_iyvbMTwXw1Juxc69yCJ2gswjE4ufuLYNw20ntNgtDmYhEBqvhHw1tQrfTgTWIhIOmZKZgBuAxYfUJ2xc84WkCv7RXqdl9x5hG_sDVFXK_NosYGqqm9rC2d3he6GUyt7seTMwDgKbW9E2Y.knnu75_B5zp7s7kbBVTar6WDatQPwsZp-aMoTYicOgg&dib_tag=se&keywords=soil+moisture+sensors&qid=1783537659&s=industrial&sprefix=soilmoisture+senso%2Cindustrial%2C185&sr=1-3-spons&sp_csd=d2lkZ2V0TmFtZT1zcF9hdGY&psc=1"> Link </a> |
| Red Button | What the item is used for | $Price | <a href="https://www.amazon.com/EGSCST-12x12x7-3-Momentary-Electronics-Prototyping/dp/B0G2CRKZ4G/ref=sr_1_1_sspa?crid=3JLYSW8IS35BU&dib=eyJ2IjoiMSJ9.K8ztKL3l65wCk2uoh4BBBJMY4zTNCQsNILMKyPbG4fcv1aglbG0GsWu5fy7YkJvGdsbj-4JNOs5i0Ulb6lptK1z7b9M13t50ydzpJ-Q5RrYPoMb5Ecpot1B95994B3IwIM1dqWakkKBk3jotqDlrhVsotBkTRgrSzPAbFsER8cHE5oGv9AGcFdAnwdcfzs57upHLnMkSZgjK9rLb53OsRxSHVAXlAkMfNkc29OmtUh_JEGJgueA-dHAzCVYgcknYKGz12dzV4pOJGfHlbT3hHxOjNsbrdDWKyrD4IEdNeD8.GgM4V9h44w9cyI7ihAawCfNbG3T-_EdFzhwA8YeWeCA&dib_tag=se&keywords=arduino%2Bbuttons&qid=1783537683&s=industrial&sprefix=arduino%2Bbutto%2Cindustrial%2C202&sr=1-1-spons&sp_csd=d2lkZ2V0TmFtZT1zcF9hdGY&th=1"> Link </a> |
| Red LED | What the item is used for | $Price | <a href="https://www.amazon.com/OWOFYDR-diffused-Electronic-Component-Indicator/dp/B09B9BYS8V/ref=sr_1_1_sspa?crid=JT82T8B7KS7S&dib=eyJ2IjoiMSJ9.4aqm9ZKwp3cxBzdYzV9Dt0oK9ZngpbtQtEEXtH13SBP5EQufjzj2rLtivxKfrd0hiehBPZePaC-Om4nHNh7989_Zv4iDnY5dhabm88QbNhHpCa_bhdjoC2pcVXH5aBCTbuZgBHTFpqjSYMCOcsTH-ZhftHDI0VCo0L350QKClNa7gLFMPGYBTJZIwWJN14Lx5kxx6QXic_-AKlStfDLt-5qv2l3d273WhIliOVWLRcwvtVLcGQACGC8aOvFEj8gbdwikOpdGudStxswlWsGrK2LTOxU98isM-SMhW8BSFwQ.MawAGQwHdIUp_EThvQT5VSKFfgdfXwW_2edflbtN5FQ&dib_tag=se&keywords=red%2Bled&qid=1783537701&s=industrial&sprefix=red%2Bled%2Cindustrial%2C203&sr=1-1-spons&sp_csd=d2lkZ2V0TmFtZT1zcF9hdGY&th=1"> Link </a> |
| Cardboard | What the item is used for | $Price | <a href="amazon.com/Flat-Cardboard-Sheets-Crafts-Corrugated/dp/B0GGRXSC68/ref=sr_1_1_sspa?crid=21V6EK16WH5V2&dib=eyJ2IjoiMSJ9.tqfGW5XW8M3C_WiJKaeDauBeGP0kmcWa_wKJp25xN6O3-MRxf9IWvFO7ozaeWC1u2sux8cpZm1T4LmBXfVA9j2Q_Ue3MbAXiTNMhBFvlMwDxoG0kmbhsnYwmNEWk1S9pf04VdrtMedIi-BRkQZa7daIwT1cz0O2H1lwGMRP68Qh5wsJns0Bhkt6P_d18SePyfdrWo8-gRqDhQTbK8sVUjcB1JDhR-QfmavILOvyBTWKlqQwK8bu8XtyUnPBYvDsAnCYBLFATzx89HY8LMzasJuJpiVfqRNf3oi3uQXwYKRo.C_9HBIIO11PvsXReo3YDT0koKHIOgHXeNixBwTFaIys&dib_tag=se&keywords=cardboard&qid=1783537727&s=industrial&sprefix=cardboard%2Cindustrial%2C216&sr=1-1-spons&sp_csd=d2lkZ2V0TmFtZT1zcF9hdGY&psc=1"> Link </a> |
| Construction Paper | What the item is used for | $Price | <a href="amazon.com/Cardstock-Construction-Invitations-Scrapbooking-Decoration/dp/B0FQ5PDG8Y/ref=sr_1_1_sspa?crid=OH1XDGZU3GQ&dib=eyJ2IjoiMSJ9.73yWsP3m367-FlAB9LyGlYZvOGvvXHOWMORMaAyb44Vx75BCOzOX8ltjQkJBINe2ykfwUAIo63E9zHQs1pIr6WZrn4onOooDBWEtVP32B1iqtPXEMfmYMaALfOokHz6n68CPCLs227g5VJctGpmpVpIDfZh6n_gMLOA7FfuMyA1BVqMbCvpFC3omN49NuEOnw1MNCLEfpXEXLH5hazNzAI5YIecPSUVbQGQSn9D06JFPbcgKpnjY_8NyD9Y1ORWb87HBBj6K-CCD-lZnzPYx2fh0fKT9y0cRbMihyUFrN3Q.2xi-fDE90ZOIbdUG1oaUg-_MG4_jDH8QrsJ7N5RLMkw&dib_tag=se&keywords=construction+paper&qid=1783537749&s=industrial&sprefix=construction+p%2Cindustrial%2C226&sr=1-1-spons&sp_csd=d2lkZ2V0TmFtZT1zcF9hdGY&psc=1"> Link </a> |
| Battery Pack | What the item is used for | $Price | <a href="https://www.amazon.com/DaierTek-Battery-Holder-Storage-Connector/dp/B09N1GDWQ9/ref=sr_1_1_sspa?crid=2NG8HIIY2LO1Y&dib=eyJ2IjoiMSJ9.BwVkKewgXHV07DVF7SWp-647jdarzT0V-pj1i-b6YQi8DuZKf1kizNw0ak7mu3H8It_ay6rjttj3hPmz37OuIuHE50gD0vUfSK5SUBwi6sOYXInmQwbEYYFIndWX7Yf64A9AqVMfPOMaQe6ATxGbIzw4S8-IET81t9ncLUf6XlqhO2-XxdfcOxHkqVzNGu3fvLcu3Hc5Uob-ALgaNbZbNEBoj7uYNpqzZjcZ_xK44WU.TmnHRcQXoEdJ8gke0SEZmRrk70lcg-sZ8J1M4nGqN88&dib_tag=se&keywords=%224%2BAA%2BBattery%2BHolder%2Bwith%2BPremium%2BJumper%2BHeader%2BWires%22%3A&nsdOptOutParam=true&qid=1783450357&sprefix=4%2Baa%2Bbattery%2Bholder%2Bwith%2Bpremium%2Bjumper%2Bheader%2Bwires%2B%2Caps%2C278&sr=8-1-spons&sp_csd=d2lkZ2V0TmFtZT1zcF9hdGY&th=1"> Link </a> |


# Other Resources/Examples
One of the best parts about Github is that you can view how other people set up their own work. Here are some past BSE portfolios that are awesome examples. You can view how they set up their portfolio, and you can view their index.md files to understand how they implemented different portfolio components.
- [Example 1](https://trashytuber.github.io/YimingJiaBlueStamp/)
- [Example 2](https://sviatil0.github.io/Sviatoslav_BSE/)
- [Example 3](https://arneshkumar.github.io/arneshbluestamp/)

To watch the BSE tutorial on how to create a portfolio, click here.

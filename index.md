# Automated Smart Bridge
The Automated Smart Bridge isn’t any normal bridge; it raises and lowers itself based off environmental conditions. When its sensors detects moisture, it displays a flood alert while simultaneously raising the bridge. The purpose of this bridge is to ensure that people can cross it safely during floody conditions.

| **Engineer** | **School** | **Area of Interest** | **Grade** |
|:--:|:--:|:--:|:--:|
| Logan W | Irvine HS | Structural Engineering | Incoming Junior

<img width="800" height="800" alt="IMG_9308 (1)" src="https://github.com/user-attachments/assets/83a671f6-715e-4569-a1fe-8958649b32cb" />

# Final Milestone

<iframe width="560" height="315" src="https://www.youtube.com/embed/B8O0gJ0z8kw?si=vknv_TfVj48AmTMz" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

Modifications
- Created an E-STOP Button that stops the bridge from moving and returns the Bridge to its neutral position
- Created two soil moisture sensors to detect water in multiple places
- Added cardboard to the structure of the bridge to make it more secure
- Added a second servo to allow an extra arm to raise the bridge
- Added an 8V battery pack to power the second servo since the Arduino's 5V can't power both servos
  
Challenges I Overcame
- Breadboard overheated when I added the 8V battery pack for the second servo
- I  unplugged all the wires and identified a dead short circuit casued by plugging the power wires of the servo into the ground rail
- I plugged my power wires into the positive rail and system functionaled normally again
  
Future Iterations
- I plan to finish the Github and Schematic

# First Milestone

<iframe width="560" height="315" src="https://www.youtube.com/embed/bIdyJx_JLe0?si=ucU84TKvC5E5MqnM" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

Different Components
- Soil Moisture Sensor: Detects moisture and sends signals to the Arduino when it detects low resistance (moist soil has low resistance compared to dry soil)
- Arduino: Acts as a motherboard and recieves signals from the Soil Moisture Sensor and sends signals to output components
- Servo: A motor that recieves signals from the Arduino and raises and lowers the bridge with its arm.
- OLED: A screen that recieves signals from the Ardunio and displays text
- Red LED: A light that recieves signals from the Arduino in Pin 6 and lights up when a flood is detected
  
Challenges I Overcame
- The physical wiring of the Arduino was the hardest part since I had to learn the basics of wiring it from scratch, now that I have finished the base project, I feel more confident about my wiring abilities
- It took trial and error, but the effort to connect the wires to the right places was incredibly rewarding once the bridge started working
  
Future Iterations
- I plan to make modifications to the bridge to improve safety

# Schematics 

<img width="881" height="705" alt="Grand Elzing-Maimu (1)" src="https://github.com/user-attachments/assets/02fe4e1d-d2a6-4e66-9b30-99c0fb837fcf" />
<img width="881" height="705" alt="IMG_6293 (1)" src="https://github.com/user-attachments/assets/895c6cc5-ba90-4340-b764-695cf1e310db" />


# Code

```c++
#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>
#include <Servo.h>

#define SCREEN_WIDTH 128
#define SCREEN_HEIGHT 64
#define OLED_RESET    -1
Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, OLED_RESET);

// Define Pins
const int ESTOP_PIN = 2;     
const int SERVO1_PIN = 4;    
const int SENSOR1_PIN = 5;   
const int LED_PIN = 6;       
const int SERVO2_PIN = 7;    // Servo 2
const int SENSOR2_PIN = 8;   

Servo servo1;
Servo servo2;

bool estopTriggered = false; 
unsigned long lastFlashTime = 0;
bool ledState = LOW;

void setup() {
  Serial.begin(9600);
  
  if(!display.begin(SSD1306_SWITCHCAPVCC, 0x3C)) {
    Serial.println(F("SSD1306 allocation failed"));
    for(;;);
  }
  
  display.clearDisplay();
  display.setTextColor(SSD1306_WHITE);
  display.setTextSize(2);
  
  pinMode(SENSOR1_PIN, INPUT);
  pinMode(SENSOR2_PIN, INPUT);
  pinMode(LED_PIN, OUTPUT);     
  pinMode(ESTOP_PIN, INPUT_PULLUP); 
  
  servo1.attach(SERVO1_PIN);
  servo2.attach(SERVO2_PIN);
  
  // STARTING POSITIONS
  servo1.write(0);    // Servo 1 starts at 0
  servo2.write(90);   // Servo 2 starts at 90 (its "closed" position)
  digitalWrite(LED_PIN, LOW);   
}

void loop() {
  if (digitalRead(ESTOP_PIN) == LOW) {
    estopTriggered = true;
  }

  display.clearDisplay();
  display.setCursor(0, 0);
  
  if (estopTriggered) {
    display.println("EMERGENCY\nSTOP!!!");
    display.display();
    
    // Snap back to safe/closed positions
    servo1.write(0);
    servo2.write(90); 
    digitalWrite(LED_PIN, HIGH); 
    
    while(true) {}
  }

  int s1 = digitalRead(SENSOR1_PIN);
  int s2 = digitalRead(SENSOR2_PIN);
  
  if (s1 == LOW || s2 == LOW) {
    display.println("FLOOD\nALERT!");
    display.display();
    
    // OPEN THE GATES
    servo1.write(90);  // Servo 1 rotates up to 90
    servo2.write(0);   // Servo 2 rotates down to 0 (which mirrors it!)
    
    if (millis() - lastFlashTime >= 200) {
      lastFlashTime = millis();
      ledState = !ledState;
      digitalWrite(LED_PIN, ledState);
    }
  } 
  else {
    display.println("Safe\nLevel");
    display.display();
    
    // KEEP GATES CLOSED
    servo1.write(0);
    servo2.write(90);
    digitalWrite(LED_PIN, LOW);
  }
  
  delay(50); 
}
```

# Bill of Materials

| **Part** | **Note** | **Price** | **Link** |
|:--:|:--:|:--:|:--:|
| Arduino Uno | Recieves inputs from Moisture Sensor and sends signals to the outputs | $14.99 | <a href="https://www.amazon.com/ELEGOO-Board-ATmega328P-ATMEGA16U2-Compliant/dp/B01EWOE0UU/ref=sr_1_2_sspa?dib=eyJ2IjoiMSJ9.f3j_Bgk5w5uiGML1DWkfl0esThnO81D4PzyHpk4VEejBGIq5HPEOCHJVyJ8VKvAZyAU3h9NlB_EOtGuanbt4cUz0bSo2Gu1lb9ian4pFVYC6LvBJ80VICkSzPfo2Lgg2fyn2PLLie0BF1ewB3_H7UqTbLZggxMaD47wMRHb40tvgxThtgf5tgNTm_uSklByC5KAOroWLvL6AHvnfcW6_-zuRdgWsU4J63RfAGXiw6Tra8DCE7hey7DmL1rJHw38_eCN5_dgITyZLBwShFUgpGg0lIme2dTNcQt1jhG3hgcE.Q_aKIgO19m6uVb-aoZkbvVjJlnQuC_-lTFjfu8EREWg&dib_tag=se&keywords=arduino+uno&qid=1783537023&s=industrial&sr=1-2-spons&sp_csd=d2lkZ2V0TmFtZT1zcF9hdGY&psc=1"> Link </a> |
| OLED Module | Displays Text | $9.99 | <a href="https://www.amazon.com/ELEGOO-Display-Compact-Self-Luminous-Projects/dp/B0D2RMQQHR/ref=sr_1_1_sspacrid=304YHUORZ27MD&dib=eyJ2IjoiMSJ9.oAZKdk9yLjCgM4o7DR9IOk7GEFPkm_nvsBW48PaUzBOpG4swxSpAAclrO102VNiVTlz9gaad0in55iUnNXdHbNmEAy9oNBKIm4xp9vNDoEcyHVgAM18CHOEURBOAAeToDiIDmlVbyYrEJgpOQI-YDFuZdXq30sLe7W8Fxm0BFzP7BwORKzBGZzGD56YtNRC0ApS-IbfGdypIo5VImYTZz3HjzS0Yndvfs7aNPBeKQo0XJiMswTEftbZbfTWz8gU3WIpABlXPn6iQXJpPexiAogsXEsiaOsTHuNKodN50U.0_b47w0BVblBmYQ7S_jrR7ngTI98cLTLfzzCfqKFOk4&dib_tag=se&keywords=oled%2Bmodule&qid=1783537269&s=industial&sprefix=oled%2Bmodu%2Cindustrial%2C186&sr=1-1spons&sp_csd=d2lkZ2V0TmFtZT1zcF9hdGY&th=1"> Link </a> |
| Breadboard | Allows for multiple wires to share a GND and 5V connection | $7.89 | <a href="https://www.amazon.com/Breadboards-Solderless-Breadboard-Distribution-Connecting/dp/B07DL13RZH/ref=sr_1_1_sspa?crid=2NP9TVEI9YJ4A&dib=eyJ2IjoiMSJ9.5Z5yTwL-oa1r18Ah_zf9OdziZtM7NtJzJKlf3z7Il1SZx-ph1l3ITB17WTZqmuHLE8rcxl4AJ3kJyvvqSFnxIw_mmmpakr_40tjpQaGTleMctk9x3ocMPnKFtmgK-heQ6kwxLurzKtJ0BVYli881oIE7HAVodJBg1jf7PQARsJucej13gxhwUeBwfHYtmZAEOkHC8t6B840zHSbLbgtBayfAElMkRCtBlJWlX0MaHD1F8PU51oWy18nfZ7PvnhNEyPKb8wYdDGHahZrcCmE7fyoPofpgdA5V0q1r_vtx21A.eieMym2UsR4nDnU1Tlh8hgEi-K1ZvfYuDblmQTrsTFo&dib_tag=se&keywords=breadboard&qid=1783537382&s=industrial&sprefix=breadboar%2Cindustrial%2C203&sr=1-1-spons&sp_csd=d2lkZ2V0TmFtZT1zcF9hdGY&psc=1"> Link </a> |
| Micro Servo | Raises and lowers the bridge | $9.88 | <a href="https://www.amazon.com/Servo-Servos-Helicopter-Airplane-Controls/dp/B0BJQ2QTHG/ref=sr_1_2_sspa?crid=5R4E6BPGGMGU&dib=eyJ2IjoiMSJ9.hpJG43Djr80Gjf-qpoo4h3_Bi-FcWHePXFoDo2a-iXlVhNWiYKO1XKWC9l0vZnSxUUsrj9FWMJeSND15Lh6sU_kWvOdWtcs_Eu313H3U_hPxl60GsbkVqacypHxKKvf-5Hug1hXBgPfE4fS1w5rqO8ZOUCz_2aAUTrTo0OZjqgD8jngG3re0oRIbFlhnMd6DLuxwMKcMrXDS3RNc1YB7-Dhq6mYGvuOI8mGqDx_EihDxR6uCy9bYiY7LILiiHfthL5R3v5VlAWF84fABHazlecP3sW9xWDnJTxVu8yujB5o.7pZH0bGyvUiyepolOOf1KzKJ4Wixm-poxHZCtpPTt3s&dib_tag=se&keywords=micro%2Bservo&qid=1783537465&s=industrial&sprefix=micro%2Bserv%2Cindustrial%2C200&sr=1-2-spons&sp_csd=d2lkZ2V0TmFtZT1zcF9hdGY&th=1"> Link </a> |
| Female Jumper Wires | Connects Moisture Sensor, OLED, and LED to the Arduino | $3.99 | <a href="https://www.amazon.com/California-JOS-Breadboard-Optional-Multicolored/dp/B0BRTJQGS6/ref=sr_1_3?crid=16LB2LRUIZU6H&dib=eyJ2IjoiMSJ9.BMvrgU_YjBIEPI70oBIcSWa3MkascqwcqGWz0yYndqacFBGJ7cfkxAuL3_nZXgYGa7nwmce8zIXo-U22N1UUaPGgNUx4T9UlCG-9_w1-P7na82losEQNgnm_eVapEifNKerimt8_QSpxABrxfmTAeUg-TN2bJRRiQlaaZbsr6Gcw-_RU7IWqXoxPpWbE4ahjK666OPJv8s_-LuNlToNpISJWUkiToG1NWe_SMrQ0PFgdvYMWhzirtFmOm-JP8HWbJPythDrZANLir3bP3CGDPT6uPlP1VpL1-TkOX_2E8d4.sggP3P7wXTfRuwIrh_hn0ynGH7fVGm91oVe1orJ-1co&dib_tag=se&keywords=male%2Bjumper%2Bwires&qid=1783537528&s=industrial&sprefix=fmale%2Bjumper%2Bwires%2Cindustrial%2C177&sr=1-3&th=1"> Link </a> |
| Male Jumper Wires | Connects the Servo and Red Button to the Arduino  | $3.99 | <a href="https://www.amazon.com/California-JOS-Breadboard-Optional-Multicolored/dp/B0BRTJQZRD/ref=sr_1_3?crid=16LB2LRUIZU6H&dib=eyJ2IjoiMSJ9.BMvrgU_YjBIEPI70oBIcSWa3MkascqwcqGWz0yYndqacFBGJ7cfkxAuL3_nZXgYGa7nwmce8zIXo-U22N1UUaPGgNUx4T9UlCG-9_w1-P7na82losEQNgnm_eVapEifNKerimt8_QSpxABrxfmTAeUg-TN2bJRRiQlaaZbsr6Gcw-_RU7IWqXoxPpWbE4ahjK666OPJv8s_-LuNlToNpISJWUkiToG1NWe_SMrQ0PFgdvYMWhzirtFmOm-JP8HWbJPythDrZANLir3bP3CGDPT6uPlP1VpL1-TkOX_2E8d4.sggP3P7wXTfRuwIrh_hn0ynGH7fVGm91oVe1orJ-1co&dib_tag=se&keywords=male%2Bjumper%2Bwires&qid=1783537528&s=industrial&sprefix=fmale%2Bjumper%2Bwires%2Cindustrial%2C177&sr=1-3&th=1"> Link </a> |
| Soil Moisture Sensor | Detects Water | $12.99 | <a href="https://www.amazon.com/MTDELE-10Pcs-Moisture-Sensor-3-3-5V/dp/B0F27WB4RK/ref=sr_1_3_sspa?crid=DZHGJCMBS7HF&dib=eyJ2IjoiMSJ9.3s60fRMlbT6tO9vQNE6Vq_7kYwZrdtP49TjwOK0S8BqeTqhP6oQ2UomhnGJL1UC22LsoNoc-O0S0aexpfMJEgo2Jnnyy41gozfQMYi4NcaQS8eoUrhYMlCTEMJ3Yo3izmsUaz0sWV5BvZ84TfB4QYPVYAhCjj_iyvbMTwXw1Juxc69yCJ2gswjE4ufuLYNw20ntNgtDmYhEBqvhHw1tQrfTgTWIhIOmZKZgBuAxYfUJ2xc84WkCv7RXqdl9x5hG_sDVFXK_NosYGqqm9rC2d3he6GUyt7seTMwDgKbW9E2Y.knnu75_B5zp7s7kbBVTar6WDatQPwsZp-aMoTYicOgg&dib_tag=se&keywords=soil+moisture+sensors&qid=1783537659&s=industrial&sprefix=soilmoisture+senso%2Cindustrial%2C185&sr=1-3-spons&sp_csd=d2lkZ2V0TmFtZT1zcF9hdGY&psc=1"> Link </a> |
| Red Button | E-Stops the Bridge | $8.99 | <a href="https://www.amazon.com/EGSCST-12x12x7-3-Momentary-Electronics-Prototyping/dp/B0G2CRKZ4G/ref=sr_1_1_sspa?crid=3JLYSW8IS35BU&dib=eyJ2IjoiMSJ9.K8ztKL3l65wCk2uoh4BBBJMY4zTNCQsNILMKyPbG4fcv1aglbG0GsWu5fy7YkJvGdsbj-4JNOs5i0Ulb6lptK1z7b9M13t50ydzpJ-Q5RrYPoMb5Ecpot1B95994B3IwIM1dqWakkKBk3jotqDlrhVsotBkTRgrSzPAbFsER8cHE5oGv9AGcFdAnwdcfzs57upHLnMkSZgjK9rLb53OsRxSHVAXlAkMfNkc29OmtUh_JEGJgueA-dHAzCVYgcknYKGz12dzV4pOJGfHlbT3hHxOjNsbrdDWKyrD4IEdNeD8.GgM4V9h44w9cyI7ihAawCfNbG3T-_EdFzhwA8YeWeCA&dib_tag=se&keywords=arduino%2Bbuttons&qid=1783537683&s=industrial&sprefix=arduino%2Bbutto%2Cindustrial%2C202&sr=1-1-spons&sp_csd=d2lkZ2V0TmFtZT1zcF9hdGY&th=1"> Link </a> |
| Red LED | Alarm | $5.99 | <a href="https://www.amazon.com/OWOFYDR-diffused-Electronic-Component-Indicator/dp/B09B9BYS8V/ref=sr_1_1_sspa?crid=JT82T8B7KS7S&dib=eyJ2IjoiMSJ9.4aqm9ZKwp3cxBzdYzV9Dt0oK9ZngpbtQtEEXtH13SBP5EQufjzj2rLtivxKfrd0hiehBPZePaC-Om4nHNh7989_Zv4iDnY5dhabm88QbNhHpCa_bhdjoC2pcVXH5aBCTbuZgBHTFpqjSYMCOcsTH-ZhftHDI0VCo0L350QKClNa7gLFMPGYBTJZIwWJN14Lx5kxx6QXic_-AKlStfDLt-5qv2l3d273WhIliOVWLRcwvtVLcGQACGC8aOvFEj8gbdwikOpdGudStxswlWsGrK2LTOxU98isM-SMhW8BSFwQ.MawAGQwHdIUp_EThvQT5VSKFfgdfXwW_2edflbtN5FQ&dib_tag=se&keywords=red%2Bled&qid=1783537701&s=industrial&sprefix=red%2Bled%2Cindustrial%2C203&sr=1-1-spons&sp_csd=d2lkZ2V0TmFtZT1zcF9hdGY&th=1"> Link </a> |
| Cardboard | Structure of the Bridge | $24.99 | <a href="https://www.amazon.com/Flat-Cardboard-Sheets-Crafts-Corrugated/dp/B0GGRXSC68/ref=sr_1_1_sspa?crid=21V6EK16WH5V2&dib=eyJ2IjoiMSJ9.tqfGW5XW8M3C_WiJKaeDauBeGP0kmcWa_wKJp25xN6O3-MRxf9IWvFO7ozaeWC1u2sux8cpZm1T4LmBXfVA9j2Q_Ue3MbAXiTNMhBFvlMwDxoG0kmbhsnYwmNEWk1S9pf04VdrtMedIi-BRkQZa7daIwT1cz0O2H1lwGMRP68Qh5wsJns0Bhkt6P_d18SePyfdrWo8-gRqDhQTbK8sVUjcB1JDhR-QfmavILOvyBTWKlqQwK8bu8XtyUnPBYvDsAnCYBLFATzx89HY8LMzasJuJpiVfqRNf3oi3uQXwYKRo.C_9HBIIO11PvsXReo3YDT0koKHIOgHXeNixBwTFaIys&dib_tag=se&keywords=cardboard&qid=1783537727&s=industrial&sprefix=cardboard%2Cindustrial%2C216&sr=1-1-spons&sp_csd=d2lkZ2V0TmFtZT1zcF9hdGY&psc=1"> Link </a> |
| Construction Paper | Road | $5.38 | <a href="https://www.amazon.com/Cardstock-Construction-Invitations-Scrapbooking-Decoration/dp/B0FQ5PDG8Y/ref=sr_1_1_sspa?crid=OH1XDGZU3GQ&dib=eyJ2IjoiMSJ9.73yWsP3m367-FlAB9LyGlYZvOGvvXHOWMORMaAyb44Vx75BCOzOX8ltjQkJBINe2ykfwUAIo63E9zHQs1pIr6WZrn4onOooDBWEtVP32B1iqtPXEMfmYMaALfOokHz6n68CPCLs227g5VJctGpmpVpIDfZh6n_gMLOA7FfuMyA1BVqMbCvpFC3omN49NuEOnw1MNCLEfpXEXLH5hazNzAI5YIecPSUVbQGQSn9D06JFPbcgKpnjY_8NyD9Y1ORWb87HBBj6K-CCD-lZnzPYx2fh0fKT9y0cRbMihyUFrN3Q.2xi-fDE90ZOIbdUG1oaUg-_MG4_jDH8QrsJ7N5RLMkw&dib_tag=se&keywords=construction+paper&qid=1783537749&s=industrial&sprefix=construction+p%2Cindustrial%2C226&sr=1-1-spons&sp_csd=d2lkZ2V0TmFtZT1zcF9hdGY&psc=1"> Link </a> |
| Battery Pack | Supplys Extra Power | $8.39 | <a href="https://www.amazon.com/DaierTek-Battery-Holder-Storage-Connector/dp/B09N1GDWQ9/ref=sr_1_1_sspa?crid=2NG8HIIY2LO1Y&dib=eyJ2IjoiMSJ9.BwVkKewgXHV07DVF7SWp-647jdarzT0V-pj1i-b6YQi8DuZKf1kizNw0ak7mu3H8It_ay6rjttj3hPmz37OuIuHE50gD0vUfSK5SUBwi6sOYXInmQwbEYYFIndWX7Yf64A9AqVMfPOMaQe6ATxGbIzw4S8-IET81t9ncLUf6XlqhO2-XxdfcOxHkqVzNGu3fvLcu3Hc5Uob-ALgaNbZbNEBoj7uYNpqzZjcZ_xK44WU.TmnHRcQXoEdJ8gke0SEZmRrk70lcg-sZ8J1M4nGqN88&dib_tag=se&keywords=%224%2BAA%2BBattery%2BHolder%2Bwith%2BPremium%2BJumper%2BHeader%2BWires%22%3A&nsdOptOutParam=true&qid=1783450357&sprefix=4%2Baa%2Bbattery%2Bholder%2Bwith%2Bpremium%2Bjumper%2Bheader%2Bwires%2B%2Caps%2C278&sr=8-1-spons&sp_csd=d2lkZ2V0TmFtZT1zcF9hdGY&th=1"> Link </a> |

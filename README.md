# 4ID3Project - READ ME TEXT

The 4ID3 Smart Cocktail Mixer 1.0 is an arduino-powered cocktail mixer created for the 4ID3 - IoT Devices and Networks course at McMaster University.

It was created by 3 fourth year students in the B.Tech. Automation Engineering Technology Program;
- [Ethan Lavis](https://www.linkedin.com/in/ethanlavis/)
- [Stephen Szulc](https://www.linkedin.com/in/stephen-szulc/)
- [Patrick Wotjera](https://www.linkedin.com/in/patrickwojtera/)

## About

The project utilizes the Arduino Cloud library, further documentation on the platform can be found **[here](https://cloud.arduino.cc/)**.

## Usage

The idea of the project is to simplify the usage process of creating mixed cocktails by inputting a simple phrase of what the user would like. Utilizing a recipe stored within the code, the user can call upon the recipe in the textbox in the Arduino Cloud dashboard. When called upon, the servo motors will open and close specific valves for a set portion of time based on the drink and mix you would like to dispense.

The following libraries are required to achieve the code followed in the main section:

```c++
#include "thingProperties.h"
#include <Servo.h>
```

The following code includes the main setup of global variables, setup function, loop function, and onMsgChange function:

```c++
unsigned long previousMillis = 0;
unsigned long previousMillisB = 0;
const long interval = 2000;
const long intervalMix = 3500;

//setup
void setup() {
  Serial.begin(9600);
  delay(1500);
  
  //set pins to servos
  vodka.attach(15);
  clubSoda.attach(16);
  orange.attach(17);
  cran.attach(18);
  
  //write initial start position to servos
  vodka.write(pos);
  clubSoda.write(pos);
  orange.write(pos);
  cran.write(pos);
  
  //Set pinMode of the Ultrasonic Sensor
  pinMode(US_TRIG_PIN, OUTPUT);
  pinMode(US_ECHO_PIN, INPUT);
  
  //Pre given from IoT Cloud Platform
  initProperties();
  ArduinoCloud.begin(ArduinoIoTPreferredConnection);
  setDebugMessageLevel(2);
  ArduinoCloud.printDebugInfo();

}

//main loop
void loop() {
  ArduinoCloud.update();
  
  //Ultrasonic sensor calculation
  long duration, Distance_Sensor;
  digitalWrite(US_TRIG_PIN,LOW);
  delayMicroseconds(2);
  
  digitalWrite(US_TRIG_PIN, HIGH);
  delayMicroseconds(10);
  
  digitalWrite(US_TRIG_PIN,LOW);
  duration = pulseIn(US_ECHO_PIN,HIGH);
  cup = (duration/2)/29.1;
  
  //for debugging
  Serial.println(duration);
  Serial.println(cup);
  
  //go back to step 0 once drink is finished
  if(msg=="Drink is made"){
    step=0;
    onMsgChange();
  }
  
  //Open "alcohol" valve
  unsigned long currentMillis = millis();
  if ((msg == "VS" || msg=="VO" || msg=="VC" || msg=="V")&& cup<=dist) {
    if (step == 0) // Open Vodka
    {
      for (pos = 0; pos <= 70; pos += 1) {
        vodka.write(pos);
        delay(1);
        if (pos == 70) {
          previousMillis = currentMillis;
          step = 1;
        }
      }
    }
    if (step == 1 && (currentMillis - previousMillis >= interval)) { // Close Vodka
      for (pos = 70; pos >= 0; pos -= 1) {
        vodka.write(pos);
        delay(1);
        if (pos == 0) {
          step = 2;
          if(msg=="V"){
            step=4;
            msg="Drink is made";
          }
        }
      }
    }
    
    //open selected "mix"
    unsigned long currentMillisB = millis();
    if (step == 2) { //Open Soda
      for (pos = 0; pos <= 70; pos += 1) {
        if(msg=="VS"){
          clubSoda.write(pos);
          delay(1);
          if (pos == 70) {
            previousMillisB = currentMillisB;
            step = 3;
          }
        }
        else if(msg=="VO"){
          orange.write(pos);
          delay(1);
          if (pos == 70) {
            previousMillisB = currentMillisB;
            step = 3;
          }
        }
        else if(msg=="VC"){
          cran.write(pos);
          delay(1);
          if (pos == 70) {
            previousMillisB = currentMillisB;
            step = 3;
          }
        }
      }
    }
    if (step == 3 && (currentMillisB - previousMillisB >= intervalMix)) {
      for (pos = 70; pos >= 0; pos -= 1) {
        if(msg=="VS"){
          clubSoda.write(pos);
          delay(1);
          if (pos == 0) {
            step = 4;
            msg = "Drink is made";
          }
        }
        else if(msg=="VO"){
          orange.write(pos);
          delay(1);
          if (pos == 0) {
            step = 4;
            msg = "Drink is made";
          }
        }
        else if(msg=="VC"){
          cran.write(pos);
          delay(1);
          if (pos == 0) {
            step = 4;
            msg = "Drink is made";
          }
        }
      }
    }
  }
}

//created by IoT cloud platform
void onMsgChange()  {
  //menu message output
  if(msg=="start" || msg== "Drink is made"){
    msg="Vodka Orange = VO --------------- Vodka Cran = VC ------------------ Vodka Soda = VS ------------------ Vodka Shot = V ---------------- ";
  }
}
```
The code includes various methods of delays (delay functions for smoother servo monitoring and utilization of the millis() library for time delays for each drink).

When a user inputs their desired drink - the following message will display in the dashboard:

```c++
 "Drink is made"
```

After this message, the user can then chose another drink to dispense only when a cup is present at the dispense tube.

## Hardware

The project utilizes various hardware components to successfully achieve the end product. The following is a quick BOM to reference:
- High Torque Servo (x4)
- ½" Ball Valves (4)
- ½" Plastic Tubing (10’ roll) (1)
- 3D printed components (4 bottle caps)
- Scrap Wood for Physical Body
- Arduino Maker 1000 (1)
- Ultrasonic Sensor (1)

## Contributing
Pull requests are welcome. For major changes, please open an issue first to discuss what you would like to change. Team members are available to be reached at the above links on LinkedIn or directly through Github.

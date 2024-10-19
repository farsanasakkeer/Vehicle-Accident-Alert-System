# Vehicle-Accident-Alert-System

## Introduction
This project is a **Vehicle Accident Alert System** that detects vehicle accidents using motion sensors and sends an emergency SMS with the location and severity of the crash. The system integrates GSM and GPS modules to communicate with emergency services in case of an accident.

## How it Works

The system continuously reads data from the accelerometer (ADXL-335). If a significant impact is detected, the system retrieves the GPS location and sends an SMS to a predefined emergency contact with the severity of the accident and the location.

## Components Used

| Component                    | Quantity |
|-------------------------------|----------|
| Arduino Nano                  | 1        |
| ADXL-335 Accelerometer         | 1        |
| GSM Module (SIM800L)           | 1        |
| GPS Module (Neo-6m)            | 1        |
| LM2596 Step-Down Converter     | 1        |
| Zero PCB Board                 | 1        |
| Buzzer (5VDC)                  | 1        |
| B3F4055 Tactile Switch         | 1        |
| SW-420 Vibration Sensor Module | 1        |

### Sim800L GSM Module

- **Operating Voltage**: 5-5.1V (powered with a lithium-ion battery and LM2596 step-down converter).
- **Antenna**: IPX antenna interface for PCB or suction cup antenna.
- **LED Indicators**:
  - **Blinking every 1 second**: Not connected to the network.
  - **Blinking every 2 seconds**: GPRS data connection is active.
  - **Blinking every 3 seconds**: Connected to the network, ready to send/receive SMS and calls.

### Neo-6M GPS Module

- **GPS**: Uses 31 satellites orbiting Earth for location and date/time transmission.
- **LED Indicator**:
  - **No blinking**: Searching for satellites.
  - **Blinking every 1 second**: Position is fixed, connected to the satellite.

### ADXL-335 Sensor

- **Voltage**: 3.7V to 5V.
- **Sensing Direction**: X, Y, and Z axes.
- **Features**:
  - Low power (350 µA).
  - Excellent temperature stability.
  - 3-axis sensing.

## Block Diagram

In this system, the ADXL-335 accelerometer detects motion or impact, and the GSM (SIM800L) module sends an SMS to the emergency contact with the accident's GPS location and intensity level.

## Circuit Diagram

- **ADXL-335**:
  - VCC → 5V
  - GND → GND
  - X-Axis → A0
  - Y-Axis → A1
  - Z-Axis → A2

- **GSM SIM800L**:
  - VCC → 5V
  - GND → GND
  - TX → D2
  - RX → D3

- **GPS Neo-6M**:
  - VCC → 5V
  - GND → GND
  - TX → D8
  - RX → D9

- **Buzzer**:
  - GND → GND
  - DO → D5

- **Push Button**:
  - GND → GND
  - DO → D6

The circuit uses an LM2596 step-down converter to power the SIM800L.

## Arduino Code

```cpp
#include <AltSoftSerial.h>
#include <TinyGPS++.h>
#include<Wire.h>
#include <math.h>
#include <SoftwareSerial.h>

const String EMERGENCY_PHONE = "+971543822440";
#define rxPin 2
#define txPin 3

SoftwareSerial sim800(rxPin, txPin);
AltSoftSerial neogps;
TinyGPSPlus gps;

String sms_status, sender_number, received_date, msg;
String latitude, longitude;
#define BUZZER 5
#define BUTTON 6
#define xPin A0
#define yPin A1
#define zPin A2

byte updateflag;
int xaxis = 0, yaxis = 0, zaxis = 0;
int deltx = 0, delty = 0, deltz = 0;
int vibration = 2;
int devibrate = 75;
int magnitude = 0;
int sensitivity = 20;
boolean impact_detected = false;
unsigned long time1, impact_time, alert_delay = 30000;

void setup() {
  Serial.begin(9600);
  sim800.begin(9600);
  neogps.begin(9600);
  pinMode(BUZZER, OUTPUT);
  pinMode(BUTTON, INPUT_PULLUP);
  
  sim800.println("AT");
  delay(1000);
  sim800.println("ATE1");
  delay(1000);
  sim800.println("AT+CPIN?");
  delay(1000);
  sim800.println("AT+CMGF=1");
  delay(1000);
  sim800.println("AT+CNMI=1,1,0,0,0");
  delay(1000);
  
  time1 = micros();
  xaxis = analogRead(xPin);
  yaxis = analogRead(yPin);
  zaxis = analogRead(zPin);
}

void loop() {
  if (micros() - time1 > 4999) Impact();
  if (updateflag > 0) {
    updateflag = 0;
    Serial.println("Impact detected!!");
    Serial.print("Magnitude:");
    Serial.println(magnitude);

    getGps();
    digitalWrite(BUZZER, HIGH);
    impact_detected = true;
    impact_time = millis();
  }
  
  if (impact_detected && (millis() - impact_time >= alert_delay)) {
    digitalWrite(BUZZER, LOW);
    sendAlert();
    impact_detected = false;
    impact_time = 0;
  }

  if (digitalRead(BUTTON) == LOW) {
    delay(200);
    digitalWrite(BUZZER, LOW);
    impact_detected = false;
    impact_time = 0;
  }

  while (sim800.available()) {
    parseData(sim800.readString());
  }
  while (Serial.available()) {
    sim800.println(Serial.readString());
  }
}

void Impact() {
  time1 = micros();
  int oldx = xaxis, oldy = yaxis, oldz = zaxis;
  
  xaxis = analogRead(xPin);
  yaxis = analogRead(yPin);
  zaxis = analogRead(zPin);

  vibration--;
  if (vibration < 0) vibration = 0;
  if (vibration > 0) return;

  deltx = xaxis - oldx;
  delty = yaxis - oldy;
  deltz = zaxis - oldz;

  magnitude = sqrt(sq(deltx) + sq(delty) + sq(deltz));
  if (magnitude >= sensitivity) {
    updateflag = 1;
    vibration = devibrate;
  } else {
    magnitude = 0;
  }
}

void parseData(String buff) {
  Serial.println(buff);
  
  unsigned int index = buff.indexOf("\r");
  buff.remove(0, index + 2);
  buff.trim();
  
  if (buff != "OK") {
    index = buff.indexOf(":");
    String cmd = buff.substring(0, index);
    buff.remove(0, index + 2);
    
    if (cmd == "+CMTI") {
      index = buff.indexOf(",");
      String temp = "AT+CMGR=" + buff.substring(index + 1) + "\r";
      sim800.println(temp);
    }
  }
}

void getGps() {
  boolean newData = false;
  for (unsigned long start = millis(); millis() - start < 2000;) {
    while (neogps.available()) {
      if (gps.encode(neogps.read())) {
        newData = true;
        break;
      }
    }
  }

  if (newData) {
    latitude = String(gps.location.lat(), 6);
    longitude = String(gps.location.lng(), 6);
  } else {
    Serial.println("No GPS data is available");
    latitude = "";
    longitude = "";
  }

  Serial.print("Latitude= "); Serial.println(latitude);
  Serial.print("Longitude= "); Serial.println(longitude);
}

String getIntensityLevel(int magnitude) {
  if (magnitude < 20) return "Minor";
  if (magnitude >= 20 && magnitude < 50) return "Moderate";
  if (magnitude >= 50 && magnitude < 100) return "Severe";
  return "Critical";
}

void sendAlert() {
  String intensityLevel = getIntensityLevel(magnitude);
  String sms_data = "Accident Alert!!\r";
  sms_data += "Intensity: " + intensityLevel + "\r";
  sms_data += "Location: http://maps.google.com/maps?q=loc:" + latitude + "," + longitude;

  sendSms(sms_data);
}

void sendSms(String text) {
  sim800.print("AT+CMGF=1\r");
  delay(1000);
  sim800.print("AT+CMGS=\"" + EMERGENCY_PHONE + "\"\r");
  delay(1000);
  sim800.print(text);
  delay(100);
  sim800.write(0x1A); // ASCII code for Ctrl+Z (end of message)
  delay(1000);
  Serial.println("SMS Sent Successfully.");
}

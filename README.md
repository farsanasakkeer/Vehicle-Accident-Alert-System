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

Note :  **SIM Compatibility**: Use a **Du SIM** for better results in the UAE.

### Neo-6M GPS Module

- **GPS**: Uses 31 satellites orbiting Earth for location and date/time transmission.
- **LED Indicator**:
  - **No blinking**: Searching for satellites.
  - **Blinking every 1 second**: Position is fixed, connected to the satellite.
GPS systems, such as the Neo-6M, generally require a clear line of sight to the sky in order to establish a reliable connection with satellites. In indoor or enclosed areas, the GPS module might struggle to get a fix on its location. This is a known limitation of GPS technology. 

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




## How to Use
- Upload the code to the Arduino Nano.
- Connect all the components as shown in the circuit diagram.
- Power on the system and wait for the device to initialize.
- In case of an accident, the system will detect the impact, get the GPS location, and send an SMS alert with the severity of the crash and the location.

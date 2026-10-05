# Component Ownership Report

### The primary purpose of this document is to establish which digital formulation for controlling each physical component.

## Entrance Phase - Component Ownership Table

| Component | Hardware Owner | Decision Owner | Controlling Programming Language | Notes |
|---|---|---|---|---|
| Ultrasonic Distance Sensor (HC-SR04) | Arduino | Rasberry Pi 3 | C++ | Senses the approaching vehicle |
| Entrance LED Light (Red/Green) | Arduino | Rasberry Pi 3 | C++ | Shows wether the vehicle may enter |
| Entrance LCD Screen | Arduino | Rasberry Pi 3 | C++ | Shows the related message |
| RFID Reader (RC522) - entrance unit, 1 of 6 | Arduino | Rasberry Pi 3 | C++ | Reads the tag; System remembers the Vehicle |
| Load Cell | HX711 | Rasberry Pi 3 | --- | Strain-gauge transducer, millivolt output, no digital interface |
| HX711 | Arduino | Rasberry Pi 3 | C++ | Amplifies and digitises the load cell segnal to measure weight |
Entrance Camera | Rasberry Pi 3 | Rasberry Pi 3 | Python | Captures vehicle details, connects to pi directly via CSI and USB |
Servo Motor | Arduino | Rasberry Pi 3 - open/close intent Arduino - abort close  | C++ | Barrier that lets Vehicle pass | 
Backup Battery | --- | --- | --- | Powers the Servo Motor in case of power cut |


## Entrance Phase - Data Ownership Table

| Data | Owner | Updated | Notes |
|---|---|---|---|
| Vehicle record (tag UID -> vehicle) | Rasberry Pi 3 | Never - UDI per vehicle is permanent | Arduino transports the UID, never interprets it |
| assigned_spot | Rasberry Pi 3 | Parking phase -- re-route only | One writer, or two vehicle get the same spot |
Spot State -- (FREE/RESERVED/HELP/OCCUPIED) | Rasberry Pi 3 | Entrance (FREE->RESERVED), parking, exit | FREE means unassigned and physically empty |

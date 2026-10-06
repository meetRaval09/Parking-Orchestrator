# Component Ownership Report

### The primary purpose of this document is to establish which digital formulation for controlling each physical component.

## Entrance Phase - Component Ownership Table

| Component | Hardware Owner | Decision Owner | Controlling Programming Language |
|---|---|---|---|
| Ultrasonic Distance Sensor (HC-SR04) | Arduino | Rasberry Pi 3 | C++ ||
| Entrance LED Light (Red/Green) | Arduino |
| Entrance LCD Screen | Arduino | Rasberry Pi 3 | C++ 
| RFID Reader (RC522) - entrance unit, 1 of 6 | Arduino | Rasberry Pi 3 | C++ |
| Load Cell | HX711 | Rasberry Pi 3 | --- |
| HX711 | Arduino | Rasberry Pi 3 | C++ |
Entrance Camera | Rasberry Pi 3 | Rasberry Pi 3 | Python |
Servo Motor | Arduino | Rasberry Pi 3 - open/close intent Arduino - abort close  | C++ |
| Breadboard | --- | --- | --- | 
Backup Battery | --- | --- | --- |


## Entrance Phase - Data Ownership Table

| Data | Owner | Updated | Notes |
|---|---|---|---|
| Vehicle record (tag UID -> vehicle) | Rasberry Pi 3 | Never - UDI per vehicle is permanent | Arduino transports the UID, never interprets it |
| assigned_spot | Rasberry Pi 3 | Parking phase -- re-route only | One writer, or two vehicle get the same spot |
Spot State -- (FREE/RESERVED/HELP/OCCUPIED) | Rasberry Pi 3 | Entrance (FREE->RESERVED), parking, exit | FREE means unassigned and physically empty |




## The "WHY" Report

### Physical Component

| Component | Why |
|---|---|
| Ultrasonic Distance Sensor (HC-SR04) | Sense the object coming from certain distance, that way system will know something coming towards the entrance |
| Entrance LED light (Red/Green) | Signal the vehicle that entrance verification complete and vehicle can enter the parking (Red -> Stop - can't procced, Green -> Clear to Procced) |
| Entrance LCD Screen | Shows the related message to the driver, only wrriten communication medium between system and driver |
| RFID Reader (RC522) - entrance unit, 1 of 6 | Tagging each vehicle that enters the parking |
| Load Cell | Connected to HX711, metal plate which helps measure weight |
HX711 | Connected to Arduino helps measuring weight of the object, purpose is to identify correct object (vehicle) allowed to enter parking |
| Entrance Camera | Capturing vehicle details (front,number plate,color) to insert vehicle records into database |
Servo Motor | It's a barrier that allows vehicle to enter parking premises , uplift 90* degree |
Breadboard | Passive wiring platform; carries connection between Arduino and components |
| Backup Battery | powers servo motor in case of power cut, prevents damage to vehicle | 
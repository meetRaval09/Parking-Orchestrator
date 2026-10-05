# Component Ownership Report

### The primary purpose of this document is to establish which digital formulation for controlling each physical component.

## Entrance Phase

| Component | Hardware Owner | Decision Owner | Controlling Programming Language | Notes |
|---|---|---|---|---|
| Ultrasonic Distance Sensor | Arduino | Rasberry Pi 3 | C++ | It sense the vehicle coming |
| Front LED Light | Arduino | Rasberry Pi 3 | C++ | Turn Green & Red |
| LCD Screen | Arduino | Rasberry Pi 3 | C++ | Shows related message |
| Reader | Arduino | Rasberry Pi 3 | C++ | System remembers the Vehicle |
| Load Cell | Arduino | Rasberry Pi 3 | C++ | Metal plate sensor |
| HX711 | Arduino | Rasberry Pi 3 | C++ | Connect with Load Cell to measure weight |
Entrance Camera | Arduino | Rasberry Pi 3 | Python <-> C++ | Capture Vehicle details |
Servo Motor | Arduino | Rasberry Pi 3 | C++ <-> Python | Barrier which lets Vehicle pass | 
Backup Battery | --- | --- | C++ | Powers Servo Motor in case of power cut |


# Component Ownership Report

### The primary purpose of this document is to establish which digital formulation for controlling each physical component.

## Parking Phase - Component Ownership Table

### Per Spot Components 

| Component | Hardware Owner | Decision Owner | Controlling Programming Language |
|---|---|---|---|
| RFID Reader (RC577) | Arduino | Rasberry Pi 3 | C++ |
| Spot LED (Red/Green) | Arduino | Rasberry Pi 3 | C++ |
| Spot LCD Screen | Arduino | Rasberry Pi 3 | C++ |
| Buzzer | Arduino | Rasberry Pi 3 | C++ |

### Camera Positioning Components

| Component | Hardware Owner | Decision Owner | Controlling Programming Language |
|---|---|---|---|
| Alignment Camera | Rasberry Pi 3 | Rasberry Pi 3 | Python |
| Stepper Motor (NEMA 17) | Arduino | For Step Generation -> Arduino, Target Spot -> Rasberry Pi 3 | C++
| Stepper Driver (A4988/DRV8825) | Rasberry Pi 3 | C++ |
| Home Endstop Switch | Arduino | Rasberry Pi 3 | C++ |
| 12v Motor Supply | --- | --- | --- |
| Rail | --- | --- | --- |
| Belt | --- | --- | --- |
| Pulleys | --- | --- | --- |
| Carriage | --- | --- | --- |
| Drag Chain | --- | --- | --- |

### Data Ownership Table

| Data | Owner | Created | Updated | Notes |
|---|---|---|---|---|
| Camera Request Queue | Rasberry Pi 3 | System Start - Empty | On <b>Request Camera ForSpot n</b>; on release or wait timeout | Bounded by 4; cannot overflow or starve |
| Camera Position / homed flag | Rasberry Pi 3 | System start - <b>NOT HOMED</b> | On homing success; on every completed move | <b>Camera Homed?</b> reads this |
| Spot -> Carriage Position Table | Rasberry Pi 3 | Build file - config file | On recalibration | Pi holds the the mm value, Arduino converts to steps |
| Vehicle status | Rasberry Pi 3 | Entrance | PArking and Exit Phase | <b>PROCESSING</b>/<b>PARKED</b>/<b>MISALIGNED</b>/<b>ALIGNMENT_UNKNOWN</b>/<b>ABANDONED</b>/<b>WAITING_FOR_SPOT</b>/<b>MAINTENANCE</b>/<b>EXIT</b>
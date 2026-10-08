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

## The "WHY" Report :

### Per Spot Component -

| Component | Why |
|---|---|
| RFID Reader (RC577) | Reads tag from vehicle at parking spot to check correct parking. |
| Spot LED (Red/Green) | Signal the vehicle upon correct parking |
| Spot LCD Screen | Showing related message |
| Buzzer | Pi-decided because the beep means "alignment off", which only the camera result established. An Arduino-local beep would sound before anything had means measured |

### Camera Positioning Components -

| Component | Why |
|---|---|
| Alignment Camera | Pi owns the wire: an 8-bit AVR has neither the bus bandwidth nor the RAM for a frame. Second of the two Pi-owned components, alogside the entrance camera. |
| Stepper Motor | Split deliberately. The Pi knows which spot needs checking; the Arduino knows how many step that is. Putting step timing on the Pi means a Linux scheduler pause mid-move, which is lost position. |
| Stepper Driver | Arduino owns STEP/DIR for the same reason - pulse timing is a real-time job. |
| Home Endstop Switch | Arduino decides, not the Pi. A carriage that must dtop at the switch cannot wait for a round trip; by the time a Pi reply arrives the belt has driven the carriage into the end plate. |
| 12 V Motor Supply | Not under software control. Listed because stepper current spikes are the loudest electrical noise in the system, and the HX711 nearby read millivolts.
| Rail, belt, pulleys, carriage, drag chain | Passive. No software reads or writes them. |

### Data -

| Data | Why this owner |
|---|---|
| Camera request queue | One writer, or two spots both believe they hold the camera and the carriage is commanded to two places at once. |
| Camera position/homed flag | One writer. A second writer's stale value makes the system trust an alignment result taken while the camera was somewhere else. |
| Spot->carriage position table | Pi owns it so recalibration edits a config file instead of reflashing firmware. The Arduino converts mm to steps; it does not decide what the mm are. |
| Vehicle atatus | Writter across three phases, so it must have exactly one owner, or a car can be <b>PARKED</b> and <b>ABANDONED</b> at the same time. |
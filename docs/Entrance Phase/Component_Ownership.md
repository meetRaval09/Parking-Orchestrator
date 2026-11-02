# Component Ownership Report

### This document records, for every physical component in the entrance phase, which device it is wired to, which device decides what it does, and which language the controlling code is written in. 

## Conventions

Throughout this document, **Pi** means the **Raspberry Pi 3 B+**, and **Arduino** means the **Arduino Mega 2560 Rev3**.

**Hardware Owner** - the device whose physical pins the component is wired to. Exactly one per component.

**Decision Owner** - the device that decides what the component should do. Exactly one per component.

**Controlling Programming Language** - the language of the code that drives the wire. Not every language in the control path.

Passive parts have no owner, because no software reads or writes them. They are listed separately below.

## Component Ownership Table

| Component | Hardware Owner | Decision Owner | Controlling Programming Language |
|---|---|---|---|
| Ultrasonic Distance Sensor (HC-SR04) | Arduino | Pi | C++ |
| Entrance LED Light (Red/Green) | Arduino | Pi | C++ |
| Entrance LCD Screen | Arduino | Pi | C++ |
| RFID Reader (RC522) - entrance unit, 1 of 4 | Arduino | Pi | C++ |
| Load Cell | HX711 | Pi | --- |
| HX711 | Arduino | Pi | C++ |
Entrance Camera | Pi | Pi | Python |
Servo Motor (Barrier) | Arduino | Pi - open/close intent<br>Arduino - abort close  | C++ |
| Breadboard | --- | --- | --- | 
Backup Battery | --- | --- | --- |

## Passive Parts

No pin, no owner, no code.

| Part | Purpose |
|---|---|
| Breadboard | Carries connections between the Arduino and the components |
| Weighing plate | Sits on the load cell. Must be longer than the vehicle so the full weight rests on it at the peak of the pass. |
| STOP HERE sign | Tells the driver where to stop. Needs no sensing and cannot fail. |
| Jumper wires, resistors, header pins | Wiring |

## Data Ownership Table

| Data | Owner | Created | Updated | Notes |
|---|---|---|---|---|
| Vehicle record (tag UID -> vehicle) | Pi | Entrance -- first read of an unknown tag | Never - UDI per vehicle is permanent | Arduino transports the UID, never interprets it |
| assigned_spot | Pi | Entrance -- when the spot is reserved | Parking phase -- re-route only | One writer, or two vehicle get the same spot |
Spot State -- (FREE/RESERVED/HELP/OCCUPIED) | Pi | system start -- all spots <b>FREE</b> | Entrance (FREE->RESERVED), parking, exit | FREE means unassigned and physically empty |


## Component Purpose

| Component | Purpose |
|---|---|
| Ultrasonic Distance Sensor (HC-SR04) | Confirms the vehicle has stopped within the stop window before the tag is read. Also acts as a watchdog on the load cell -- an object in the lane for more than 10 s with no weight event means the load cell has failed |
| Entrance LED light (Red/Green) | Signals whether the vehicle may enter. red -- stop, cannot proceed. Green -- clear to proceed |
| Entrance LCD Screen | Shows the related message to the driver. The only wrriten communication medium between system and driver |
| RFID Reader (RC522) - entrance unit, 1 of 4 | Reads the tag of each vehicle entering the car park. Supplies identity; the render that sees the tag also supplies location |
| Load Cell | Detects vehicle arrival and measures gross weight as the vehicle rolls across the plate. A strain-gauge transducer with no digital interface -- its four wires terminate at the HX711, never at the Arduino |
HX711 | amplifies and digities the load cell signal. RATE pin tied high for 80 samples per second; the default 10 SPS is too slow to catch the peak of a rolling pass |
| Entrance Camera | Capturing vehicle details (front,number plate,color) to insert vehicle records into database |
Servo Motor (Barrier) | It's a barrier that allows vehicle to enter parking premises , uplift 90* degree |
Breadboard | Passive wiring platform; carries connection between Arduino and components |
| Backup Battery | powers servo motor in case of power cut, prevents damage to vehicle |

# Why These Owners 

## Principles

### 1. One owner per wire. Every component is wired to exactly one device. Where that device is not the Arduino, the row says so and the reason is given below.
### 2. Decision live where the data lives. Any component whose behaviour depends on <b>assigned_spot</b> or spot state is decided by the Pi, because only the Pi holds those.
### 3. Reflexes live where the latency is. A reaction that must happen in milliseconds stays on the Arduino, even when the Pi owns the intent behind it.
### 4. Perception on the Pi, GPIO on the Arduino. Anything producing a frame is the Pi's anything producing or reading a logic level is the Arduino's.

## Per component 

| Component | Why this owner |
|---|---|
| Ultrasonic Distance Sensor | The Arduino reports a raw distance. If it also judged "in the window", the threshold would exist in two places and could disagree |
| Entrance LED Light | Single writer. The LED asserts teh entry decision; a second writer could show green while the barrier stays shut |
| Entrance LCD Screen | Single writer. Two senders interleaving on one display produce a half-overwritten message, and a driver acting on half a message does the wrong thing |
| RFID Reader | The Arduino forwards the UID unaltered. Interpreting it needs the vehicle record, which is on the Pi -- deciding at the gate would require a second copy of the database, and two copies disagree |
| Load Cell | Owned by the HX711, not the Arduino. It outputs a few millivolts and has no digital interface, so it cannot be wired to a microcontroller pin at all |
| HX711 | The Arduino owns DT/SCK because the read is a timing-sensitive bit shift. A Linux scheduler pause mid-read on the Pi would corrupt the sample |
| Entrance Camera | The only component the Pi owns at the wire. An 8-bit AVR has neither the bus bandwidth nor the RAM to hold a frame |
| Servo Motor | Split deliberately. The Pi owns who may enter, because that answer lives in the database. The Arduino owns the close, because a barrier that waits for a round trip is a barrier that closes on a car |
| Backup Battery | Not under software control. Listed because the servo's stall current is the largest draw in the system, and an undocumented power source is an undocumented failure mode |
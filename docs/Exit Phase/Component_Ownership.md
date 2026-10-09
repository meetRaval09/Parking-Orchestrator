# Component Ownership Report

### The primary purpose of this document is to establish which digital formulation for controlling each physical component.

## Physical Component -

| Component | Hardware Owner | Decision Owner | Controlling Programming Language |
|---|---|---|---|
| RFID Reader tag (RC522) | Arduino | Rasberry Pi 3 | C++ |
| Exit Servo Motor | Ardiono | Rasberry Pi 3 | C++ |
| Exit Gate Clearance Sensor (IR break-beam) | Arduino | Arduino | C++ |
| Exit Screen | Arduino | Rasberry Pi 3 | C++ |

## Data -

| Data | Owner | Created | Updated | Notes |
|---|---|---|---|---|
| Vehicle Status | Rasberry Pi 3 | Entrance | Set to <b>EXIT</b> on clearance, not on tag read | Defined once in the Entrance phase table |
| Spot state | Rasberry Pi 3 | System start - <b>FREE</b> | Released to <b>FREE</b> on clearance | Defined once in the Entrance phase table |

## The "WHY" Report :

| Component | Why this owner |
|---|---|
| Exit RFID Reader | The Arduino transports the UID; only the Pi holds the record says whether this vehicle may leave. Deciding at the gate would need a second cpy of the vehicle Database. |
| Exit Barrier Servo | Split. The Pi owns who may leave, because that answer lives in the database. The Arduino owns the close, because a barrier that waits for a network round trip is a barrier that close on a car. |
| Exit Gate Clearance Sensor | Arduino decides. Its whole purpose is a relation too fast to route through the Pi. It reports the event afterwards; it does not ask permission. |
| Exit Screen | Single writer. A refusal message and a status message overwriting each other leaves the driver with neither. |

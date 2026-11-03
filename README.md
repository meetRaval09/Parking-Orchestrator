# Parking Orchestrator

## A four-bay automated car park, built as a physical bench model. RFID identity, weight-based vehicle classification, and camera-verified parking alignment.



---

## What it does

A car arrives, is weighed in motion, identified by its tag, and sent to a spot.
A camera on a moving rail checks it parked inside the lines. The spot is released when it drives out.

Three details that shaped the design :

- **Weight classifies before position is checked** -- so a pedestrian never ties up the gate
- **One camera serves four spots** from a linear stage, with queued access and a service timeout
- **A spot is freed on physical departure, not on a tag read** -- presenting a tag proves presence, not exit

---

## How it's built

| Raspberry Pi 3 Model B+ | Arduino Mega 2560 Rev3 |
|---|---|
| Decisions, database, vision | Every pin, and reflexes too fast to send upstream |

Linked over serial at 115200 baud with a checksummed ASCII protocol.

---

## Documentation

**Entrance Phase** --> [Flowchart](docs/Entrance%20Phase/Flowchart.drawio) . [Component_Ownership](docs/Entrance%20Phase/Component_Ownership.md) . [Communication_Protocol](docs/Entrance%20Phase/Communication_Protocol_Report.md) . [State_Machine](docs/Entrance%20Phase/State_Machine_Report.md) . [Pin_Mapping](docs/Entrance%20Phase/Pin_Assignment_Map_Report.md)

**Parking Phase** --> [Flowchart](docs/Parking%20Phase/Flowchart.drawio) . [Component_Ownership](docs/Parking%20Phase/Component_Ownership.md) . [Communication_Protocol](docs/Parking%20Phase/Communication_Protocol_Report.md) . [State_Machine](docs/Parking%20Phase/State_Machine_Report.md) . [Pin_Mapping](docs/Parking%20Phase/)

**Parking Phase** --> [Flowchart](docs/Exit%20Phase/Flowchart.drawio) . [Component_Ownership](docs/Exit%20Phase/Component_Ownership.md) . [Communication_Protocol](docs/Exit%20Phase/Communication_Protocol_Report.md) . [State_Machine](docs/Exit%20Phase/State_Machine_Report.md) . [Pin_Mapping](docs/Exit%20Phase/)


---

## Hardware

Every part, all three phases -- view or download : **[hardware.csv](hardware.csv)**

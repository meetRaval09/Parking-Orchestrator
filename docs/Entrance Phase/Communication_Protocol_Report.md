# Communication Protocol Report 

## Overview -

### Three links carry every message in this phase :

| No. | Link | Physical | Role |
|---|---|---|---|
| 1 | Rasberry Pi <--> Arduino Mega | USB serial (CDC) 115200 8N1 | Commands down, events up. The only link between the two tiers |
| 2 | Arduino <--> Components | SPI, I<sup>2</sup>C, digital, PWM | Device-level buses |
| 3 | Pi orchestrator <--> Pi camera process | Unix domain socket | keep a camera stall out of the control loop |

### Link 1 is the one that needs a designed protocol. Links 2 and 3 use published or trivial interfaces and are documented for completeness.
# Communication Protocol Report 

## Overview -

### Three links carry every message in this phase :

| No. | Link | Physical | Role |
|---|---|---|---|
| 1 | Rasberry Pi <--> Arduino Mega | USB serial (CDC) 115200 8N1 | Commands down, events up. The only link between the two tiers |
| 2 | Arduino <--> Components | SPI, I<sup>2</sup>C, digital, PWM | Device-level buses |
| 3 | Pi orchestrator <--> Pi camera process | Unix domain socket | keep a camera stall out of the control loop |

### Link 1 is the one that needs a designed protocol. Links 2 and 3 use published or trivial interfaces and are documented for completeness.

# Link 1 - Pi <--> Arduino

## 1.1 Physical and framing -

| Property | Value | Why |
|---|---|---|
| Transport | USB serial via then Mega's ATmega16U2 | One cable for power, data and reflashing |
| Baud | 115200 | ~11.5lB/s; a 40-byte frame takes = 3.5 ms |
| Format | 8N1, no flow control | Frames are short; no buffer pressure at this rate |
| Encoding | ASCII, line-oriented | Readable in a serial monitor with no decoder |

### Why ASCII and not binary. Binary would be perhaps four times smaller. At 115200 baud with a few frames per second, that saving is worth nothing, and it costs the ability to open a serial monitor during a demo and read what the system is saying. For a bench system, debuggability beats effciency - and the bandwidth headroom makes it a free choice rather than a compromise.
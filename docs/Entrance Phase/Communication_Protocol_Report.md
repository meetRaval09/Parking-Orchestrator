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

# 1.2 Frame format -

| Field | Format | Notes |
|---|---|---|
| <b>#</b> | literal | Start delimiter. Anything before it is discarded. |
| <b>seq</b> | 1-999, decimal | Wraps at 999. Matches a reply to its command and detects duplicates. |
| <b>VERB</b> | 3-6 uppercase letters | Message type |
| <b>aeg</b> | comma-separated, no spaces | Verb-specific |
| <b>*</b> | literal | End of checksummed region |
| <b>chk</b> | 2 hex digits | XOR of every byte between <b>#</b> and <b>*</b>, exclusive |
| <b>\n</b> | 0x0A | Terminator |

### The XOr-between-delimiters checksum is the NMEA 0183 scheme-worth naming in the report, because adopting an established format rather than inventing one is itself a defensible decision.

## Examples :

# 042,LED,ENT,GREEN*3c       Pi --> Arduino
# 042,ACK*7A                 Arduino --> Pi
# 118,TAG,ENT,A42F9103*5E    Arduino --> Pi
# 118,ACK*41                 Pi --> Arduino
# 043,SERVO,ENT,OPEN*12      Pi --> Arduino
# 043,NAK,BUSY*08            Arduino --> Pi

# 1.3 Message catalogue - entrance phase 

## Pi --> Arduino (commands)

| Verb | Args | Effect |
|---|---|---|
| <b>LED</b> | <b>ENT</b>, <b>RED</b> / <b>GREEN</b> / <b>OFF</b> | Set entrance LED |
| <b>LCD</b> | <b>ENT</b>, <b><text></b> | Write entrance screen (text <= 32 chars, commas escaped as <b>\;</b>) |
| <b>SERVO</b> | <b>ENT</b>, <b>OPEN</b> / <b>CLOSE</b> | Command the barrier |
| <b>TARE</b> | --- | Zero the HX711 |
| <b>CFG</b> | <b><param></b>, <b><values></b> | Set a threshold (e.g. <b>CFG,DIST_TRIG,12</b>) | 
| <b>SYNC</b> | --- | Request full device state; used after a Pi restart |
| <b>PING</b> | --- | Liveness check |


## Arduino --> Pi (events)

| Verb | Args | Meaning | State machine event |
|---|---|---|---|
| <b>DIST</b> | <b><cm></b> | Ultrasonic reading, on threshold crossing | E1 / E2 |
| <b>SETTLED</b> | <b>ENT</b> | Distance stable at read position for T<sub>1</sub> | E3 |
| <b>TAG</b> | <b>ENT</b>,<b><uid hex></b> | Tag read at the entrance reader | E4 |
| <b>NOTAG</b> | <b>ENT</b> | Read window elapsed with no tag | E5 |
| <b>WGT</b> | <b><grams></b> | Weight reading | - (feeds E7) |
| <b>BEAM</b> | <b>ENT</b>, <b>BROKEN</b> / <b>CLEAR</b> | Clearance sensor edge | E11a / E11b |
| <b>SERVO</b> | <b>ENT</b>, <b>OPENED</b> / <b>CLOSED</b> | Barrier reached position | E13 |
| <b><b>FAULT</b> | <b><device></b>, <b><code></b> | Device unavailable | E16 |
| <b>BOOT</b> | <b><fw></b> | Arduino started or reset | - (triggers <b>SYNC</b>) |

### That last colemn is the point of the document. Every state machine event has exactly one protocol message that produces it, and every message maps to a state machine event or is explicitly marked as feeding one. A message with no event is dead protocol; an event with no message is a transition nothing can trigger. Both are bugs, and the table is how you find them.

## Two entrance events have no message, deliberately :

| Event | Source | Why not on this link |
|---|---|---|
| E9 <b>spot_reserved</b>, E10 <b>no_spot_free</b>, E14 <b>spot_became_free</b> | Pi- internal | Spot state lives on the Pi. These never cross the link. |
| E15<b>staff_clear</b> | Operator | Out of band |
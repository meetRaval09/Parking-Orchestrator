# Pin Assignment Map -- Entrance Phase

## Convention 

### Throughout this document, Pi means the Rasberry Pi Model B+, and Arduino means the Arduino Mega 2560 Rev3

### Pin numbers are the Arduino digital pin numbers printed on the board, not the ATmega2560 port names.

### A pin is listed once. If a component appears in more than one phase, it is allocated in the phase that owns it and cross-referenced elsewhere.

# Reserved System-Wide

## These are not allocatable to any phase.

| Pin | Signal | Why it's fixed |
|---|---|---|
| 0, 1 | USB serial to the Pi | <b>Serial0</b>. Using these as GPIO breaks the link and the upload |
| 50 | SPI MISO | Hardware SPI, wired inside the chip |
| 51 | SPI MOSI | Hardware Pi |
| 52 | SPI SCK | Hardware SPI. Not the HX711's SCK |
| 53 | SPI SS | Unused as a chip select, but must be set <b>OUTPUT</b> in <b>setup()</b> or the Mega drops out of SPI master mode |
| 49 | RC522 RST | One reset line shared by all six readers |
| 20 | I<sup>2</sup>C SDA | Entrance LCD |
| 21 | I<sup>2</sup>C SCL | Entrance LCD |

### Pins 44, 45, 46 -- leep free of PWM use. The servo library claims Timer5 as soon as any servo is attached, which disables <b>analogWrite()</b> on those three.

## Entrance Phase Allocation 

| Pin | Dir | Device | Signal | Logic | Notes |
|---|---|---|---|---|---|
| 11 | OUT | Barrier servo | PWM | % V | PWM pin; Servo lobrary, Timer5 |
| 22 | OUT | RC522 (entrance) | CS | 3.3 V | Level shift required |
| 24 | OUT | Ultrasonic -- stop window | TRIG | 5 V | 10 µs pulse |
| 25 | IN | Ultrasonic -- stop window | ECHO | 5 V | <b>pulseIn</b>, 30 ms timeout |
| 26 | OUT | Entrance LED | Red | 5 V | 220Ω series resistor |
| 27 | OUT | Entrance LED | Green | 5 V | 220Ω series resistor |
| 28 | IN | HX711 | DT | % V | --- |
| 29 | OUT | HX711 | SCK | % V | Bit-banged. Not SPI |
| 30 | OUT | Ultrasonic -- post-barrier | TRIG | 5 V | See the note below |
| 31 | IN | Ultrasonic -- post-barrier | ECHO | 5 V | --- |

## Interrupt Budget -- System-Wide 

### The Mega has four usable external-interrupt pins once I<sup>2</sup>C has taken 20 and 21. This is the tightest resource in the build -- tighter than pin count, which has thirty-odd spare.

| Pin | INT | Use | Phase | status |
|---|---|---|---|---|
| 2 | INTO | Entraqnce clearance | Entrance | Free, pending the beam decision |
| 3 | INT1 | Exit clearance beam  | Exit | Allocated |
| 18 | INT5 | Atepper home endstop | Parking | Allocated |
| 19 | INT4 | Spare | --- | Free |


# Two Traps 

## 1. The RC522 is a 3.3 V part on a 5 V board.

### <b>VCC</b> goes to the Mega's 3.3 V rail, never 5 V -- that kills the module immediately. Its inputs (<b>MOSI</b>,<b>SCK</b>,<b>CS</b>,<b>RST</b>) are not reliably 5 V tolerant either. Some modules survive direct connection, some don't, and the ones that fail do it intermittently weeks later.

### Use a 4-channel bidirectional level shifter, or 1 kΩ series resistor in each line from the Mega to the reader. <b>MISO</b> needs nothing -- the reader drives the Mega, and 3.3 V is a valid logic high for a 5 V AVR input.

## Two different signals called SCK 

### Pin 52 is the SPI clock. Pin 29 is the HX711's clock. They are unrelated and must not be joined. Label both wires at both ends.

# Totals

|    | Count |
|---|---|
| Reserved system-wide | 8 |
| Entrance phase | 10 |
| Running total used | 18 of 54 digital |
| Interrupt pins used by this phase | 0, or 1 if the beam replaces the post-barrier ultrasonic |

## Analog pins A0-A15 are untouched.
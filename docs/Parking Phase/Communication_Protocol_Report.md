# Parking Phase -- Communication Protocol

### Same three links, same frame format. This section documents only what differs, plus the one structural addition the phase forces; a protocol for operations that the seconds rather than milliseconds.

# What's new in this phase 

| Points | Entrance | Parking |
|---|---|---|
| Addressed devices | one of each | four of each -- every message carries a spot index |
| Longest operation | servo sweeps, ~0.5 s | carriage move, ~3.5 s |
| SPI device on the bus | 1 reader | 4 readers + 4 screens |
| Verdict source | Arduino | Pi camera process -- crosses a diferent link |
| Electrical noise | low | stepper current spikes |

# Link 1 -- Pi <--> Arduino 

# 1.1 Two-Phase commands

### Everything in the entrance phase completes faster than a human notices, so command -> ack was enough. the carriage move does not. Blocking the link for 3.5 seconds waiting for an ack would stall every reader, screen and LED in the system.

### So long operations use three messages, not two:

|---|---|---|
| #201, MOVE, 310*4E | Pi --> Arduino | Command |
| #201, ACK, Accepted*1B ~5 ms | Arduino --> Pi | "I have it, I am moving" |
| #201, MOVED, 310*22 ~3.5 s | Arduino --> Pi | "I arrived" |

### <b>ACK</b> means accepted, never finished. The completion event carries the same <b>seq</b>, so the Pi can match it to the original command even after other traffic has passed.

### Rule for the report : any operation that canot complete inside the 100 ms command timeout is two-phase. In this system that is exactly two commands -- <b>MOVE</b> and <b>HOME</b>/ Everything else is single-phase.

### This is also why the camera queue lives on the Pi rather than the Arduino. The Arduino can execute one move; it has no idea three other spots are waiting. Arbitration belongs where the state is.

# 1.2 Message catalogue -- Parking phase 

## Pi --> Arduino

| Verb | Args | Phase | Effect |
|---|---|---|---|
| <b>LED</b> | <b><spot 1-4></b>, <b>RED / GREEN / OFF</b> | single | Spot LED |
| <b>SCR</b> | <b><spot></b>, <b><text></b> | single | Spot screen, <= 32 chars |
| <b>BUZ</b> | <b><spot></b>, <b>ON, <period_ms> / OFF</b> | single | <b>BUZ,3,ON,2000</b> = beep every 2 s | 
| <b>MOVE</b> | <b><pos_mm></b> | two | Move carriage to absolute position |
| <b>HOME</b> | --- | two | Run homing routing |
| <b>STOP</b> | --- | single | Abort motion immediately | 
| <b>SYNC</b> | --- | single | Report all device state |

## Arduino --> Pi 

| Verb | Args | Meaning | State machine event |
|---|---|---|---|
| <b>TAG</b> <b><spot></b>, <b><uid></b> | Tag read at a spot | P1 / P2 / P3 -- the Pi decides which |
| <b>MOVED</b> | <b><pos_ms></b> | Carriage arrived | C-T5 <b>MOVING -> CHECKING</b> | 
| <b>MOVEFAIL</b> | <b><reason></b> | Move aborted | C-T7 -> <b>DISABLED</b> |
| <b>HOMED</b> | <b>0</b> | Endstop found, position aeroed | C-T2 <b>HOMING -> IDLE</b> |
| <b>HOMEFAIL</b> | <b>TIMEOUT</b> | Endstop not found | C-T3 -> <b>DISABLED</b> |
| <b>Endstop</b> | <b>HIT</b> | Reflex fored, motor stopped | reported, not requested |
| <b>FAULT</b> | <b><devices></b>, <b><code></b> | Device unavailable | P12 |

### One row deserves attention. <b>TAG, <spot>, <uid></b> is a single message type that produces three dfferent state machine events. The Arduino does not know whether a UID belongs at that spot -- that requires <b>assigned_spot</b>, which lives on the Pi. So the Arduino report the bare fact and Pi classifies it:

| Condition on the Pi | Event |
|---|---|
| <b>spot == assigned_spot</b> and <b>uid == this vehicle</b> | P1 |
| <b>spot != assigned_spot</b> and <b>uid == this vehicle</b> | P2 |
| <b>spot == assigned_spot</b> and <b>uid != this vehicle</b> | P3 |

### This is the ownership rule showing up as protocol: the Arduino transports the UID, it never interprets it. The message format makes that structural rather than aspirational -- there is no field in which the Arduino could express an opinion.

### <b>ENDSTOP, HIT</b> is the other one to note. It is sent after the Arduino has already stopped the motor. It is a report, not a request for permission. The protocol has no message by which the Pi could authorise a stop, and absence is the design.

## 1.3 Why step pulses are not on this link 

### At 1/6 microstepping with a 20-tooth GTT2 pulley. one step is 0.0125 mm. A 500 mm move is 40000 steps, and at 150 mm/s that's 12000 steps per sencond.

### The link carries roughly 300 frames per sencond at 115200 vaud. Step generation over serial is short by a factor of forty, before accounting for jitter -- and a stepper fed irregular pulses loses position silently.

### So the Pi sends a destination in millimetres and the Arduino generates every pulses locally. The ownership document says the Arduino owns step generation; 

# 1.4 Noise

### Stepper coil current swithes hard, and the motor leads run near the SPI bus. This shows up at the protocol layer as checksum failures that correlate with carriage movement -- which is a confusing symptom unless you are expecting it.

| Miligation | --- |
|---|---|
| Motorleads routed away from SPI and HX711 lines, crossing at night angles where unavoidable | wiring |
| Separate 12 V supply, grounds tied at one point only | power |
| SPI at 1 MHz | already chosen for cable length; hels here too 
| Log checksum-failure counts againts carriage state | diagnostics |

### That last one is cheap and worth building in. If failure cluster during <b>MOVING</b>, it's coupling; if they are uniform, it's cable length. One counter separates two causes that look identical from the outside.

# 1.5 SPI bus contention -- the subtle one

### Four spot reader and four spot screens shares one SPI bus with one chip-select each, and the screens <b>DC</b> and <b>RES</b>. Only one device can be selected at a time, so the Arduino must serialise every transaction.

### A full 32-character screen update is not instant. While it is in progress, no reader is being polled. a tag presented in that window can be missed.

| Mitigation | --- |
|---|---|
| Write screens in chunks, polling readers between chunks | firmware structure |
| Never redraw a screen whose text has not changed | the comman case is a repeated identical write |
| Accept the miss -- the reader is polled continuously and a car does not leave in 20 ms | reality check |

### the third point is honest one and probably sufficient at this scale. Worth starting anyway: a reviewer who knows SPI will wonder whether you noticed, and "noticed, measured, judged acceptable" reads very differently from silence.

# Link 3 -- Pi orchestrator <--> camera process

## Extended for alignment.

## <b>Request</b>:

### {"op":"check-alignment","spot":3,"timeout_ms":1500}

## <b>Response</b>:

### {"ok":true,"aligned":false,"offset_mm":[18.4,-6.1],"angle_deg":7.2,"frame":"/var/parking/img/spot3_1729.jpg"}

| Field | Use |
|---|---|
| <b>aligned</b> | Produces P13 or P14 |
| <b>offset_ms</b>, <b>angle_deg</b> | Logged, not acted on -- but they are what makes a repeatability claim measurable |
| <b>frame</b> | Saved for debugging a disputed verdict |
| <b>ok:false</b> | Produces P12 <b>camera_unavailable</b> |

### The spot boundary polygons live in a Pi config file, owned by the orchestrator and passed to the camera process at startup. Recalibration edits that file; neither the Arduino nor the firmware is involved. That matches at data Ownership row for the spot -> position table, and for the same reason.

# 2. Timing budget, and what it forces

| Step | Time |
|---|---|
| <b>MOVE</b> command --> <b>ACK</b> | ~5 ms |
| Carriage traverse, worst case across the deck | ~3.5 s |
| <b>MOVED</b> event | ~2 ms |
| Alignment check on a Pi 3 | 200-400 ms |
| One camera service, first attempt | = 4 s |
| Driver repositions between attempts | 3-5 s (human) |
| One seesion, five attempts, worst case | = 45 s |

### With four spot, a seesion at the back of the queue could wait three full session ahead of it -- over two minutes -- and your <b>Camera Wait Timeout</b> would fire long before the camera ever reached it. Every queued car would land in <b>ALIGNMENT_UNKNOWN</b>, and queue would look broken when in fact it was working exactly as drawn.

### Two numbers have to be set together :

| Parameter | Proposed | Reasoning |
|---|---|---|
| Max session hold | 30 s wall clock, in addition to the 5 attempt limit | Caps one car's claim on a shared resources regaedless of how slowly the driver moves |
| Camera wait timeout | 120 s | 3 session ahead * 30 s, plus margin. Must exceed worst-case queue service or the back of the queue always loses. |

### The rule : a wait timeout on a shared resources must be longer than the worst-case time to serve everyone ahead of you. Set it shorter and you have not built a queue -- you have built a queue that quietly discards its tail.

### The 30 s hold is also a new transition your parking mahine does not have: <b>CHECKING_ALIGNMENT</b> / <b>REALIGNING</b> --> <b>AWAITING_STAFF</b> on hold expiry, with the same camera release as P-T20. Worth adding -- right now only the attempt counter can end a seesion, and a driver who never moves never increments it.

### A car waiting two minutes for verification is fine, incidentally. It is already parked. The only cost is a delayed green LED.

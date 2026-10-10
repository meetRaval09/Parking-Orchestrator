# State Machine

## Overview -

### The entrance phase is governed by one controller state machine - the <b>Entrance Gate</b> - which owns the sequence from a vehicle approaching to the barrier closing behind it. It runs on the Rasberry Pi 3. 

### Two other machines are touched but not owned here. The <b>Spot</b> machine makes its <b>FREE -> Reserved</b> transition on a command from this phase, and the <b>Vehicle</b> machine is created by this phase and handed on the parking phase. Both are defined in their own section; this document names only the transition the entrance causes.

### Exactly one Entrance Gate machine exist.

# 1. State -

| State | Meaning -> "right now the gate is..." |
|---|---|
| <b>IDLE</b> | Nothing at the gate. Barrier closed, LED red, Screen shows the default message. |
| <b>APPROACHING</b> | Something has beedn detected in the lane but has not settled at the read position. |
| <b>READING_TAG</b> | A vehicle is stopped at the reader. The system is attempting to read a tag. |
| <b>IDENTIFYING</b> | A tag UID has been read. Camera capture and weight measurement are in progress; the record is being created or found. |
| <b>ASSIGNING</b> | The vehicle is known. A free spot is being selected and reserved. |
| <b>OPENING</b> | A spot is reserved. The barrier has been commanded open; LED green; screen shows the spot number. |
| <b>WAITING_CLEARANCE</b> | Barrier open, vehicle passing through. |
| <b>CLOSING</b> | Clearance confirmed. The barrier is being closed. |
| <b>REFUSED_UNREADABLE</b> | No tag could be read within the attempt limit. Barrier shut, staff notified. |
| <b>REFUSED_FULL</b> | The vehicle is identified but no spot is free. Barrier shut, screen shows <b>Car Park Full</b>. |


### <b>Initial state</b>: <b>IDLE</b>, entered at system start after initialisation.

### <b>Terminal state</b>: none. The gate cycles indefinitely.

### <b>Why two refusal states and not one</b>

#### <b>RESUSED_UNREADABLE</b> and <b>REFUSED_FULL</b> look like the same situation - barrier shut, car waiting - but they leave by different doors. <b>REFUSED_FULL</b> resolves on its own when any spot becomes <b>FREE</b>. <b>REFUSED_UNREADABLE</b> cannot resolve without a person.

#### <b>Rule applied here, and worth stating in the report:</b> if two situation leave by the same exit, they are one state with a reason field. If they leave by different exits, they are different states. Collapsing them would hide the fact that one is self-healing and one is not.

### <b>Why <u>DERADED</u> isnot a state -

#### A load cell or camera fault does not change what the gate is doing - it changes how thoroughly it checks. Modelling it as a state would require a degraded twin of every state above, doubling the machine for no new behaviour.

### It is therefore a mode flag, not a state -

| Flag | Set When | Effect |
|---|---|---|---|
| <b>WEIGHT_AVAILABLE</b> | false on HX711 fault | <b>IDENTIFYING</b> skips the weight step; record flagged <b>WEIGHT_UNKNOWN</b> |
| <b>IMAGE_AVAILABLE</b> | false on camera fault | <b>IDENTIFYING</b> skips capture; record flagged <b>IMAGE_UNKNOWN |

### Both flags are owned by the Pi and cleared only by a successful read after a fault

<hr>

# 2. Events -

| ID | Event | Source |
|---|---|---|
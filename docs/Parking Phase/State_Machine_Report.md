# State Machine Report 

## Overview -

### The Parking Phase is governed by one system controller state machine - in parking phase; internal  system will control the entire flow, from vehicle entering to the parking premises to parking process on the designated spot.

## 1. Event - 

| State | Mesning -> "right now the gate is..." |
|---|---|
| <b>IN_TRANSIT</b> | Past the barrier, moving towards its assigned park area. Nothing has reported it yet. |
| <b>STALLED_IN_LINE</b> | Stopped in the lane without reaching the spot. Breakdown timer running. |
| <b>WRONG_SPOT</b> | A spot read its tag, but not the assigned one. Grace period running. |
| <b>RE_ROUTING</b> | Its assigned spot has reported a different vehicle's tag. A new spot is being sought. |
| <b>WAITING_FOR_SPOT</b> | Held in the lane because no spot is free. Staff notified. |
| <b>AWAITING_CAMERA</b> | At the correct spot, queued for the camera. |
| <b>CHECKING_ALIGNMENT</b> | Camera in position over the spot, evaluating. |
| <b>REALIGNING</b> | Alignment failed. Buzzer sounding, driver adjusting. |
| <b>AWAITING_STAFF</b> | Five attempts exhausted, Screen shows <b>Please Wait</b>, one-minute timer running. |
| <b>PARKED</b> | Terminal. Aligned and verified. |
| <b>MISALIGNED</b> | Terminal. Staff handling, spot stays held. |
| <b>ALIGNMENT_UNKNOWN</b> | Terminal. Physically parked, never verified. |
| <b>ABANDONED</b> | Terminal. Vehicle left during the staff wait; spot released. |
| <b>MAINTENANCE</b> | Terminal. Towed to the repair spot. |

### Initial State : <b>IN_TRANSIT</b>, entered when the entrance gate reaches <b>CLOSING -> IDLE</b>.

### Terminal States : five. Nine states are live; the rest are exits.

## <b>Why five terminal states and not one</b>

### They differ in what is true afterwards, and that is what a terminal state records:

| Terminal | Bay afterwards | Vehicle afterwards | Need a staff? |
|---|---|---|---|
| <b>PARKED</b> | <b>OCCUPIED</b> | In the car parked | No |
| <b>MISALIGNED</b> | <b>HELD</b> | In the car park | Yes |
| <b>ALIGNMENT_UNKNOWN</b> | <b>OCCUPIED</b> | Inthe car park | No |
| <b>ABANDONED</b> | <b>FREE</b> | Gone | No |
| <b>MAINTENANCE</b> | <b>FREE</b> | In the repair spot | Yes |

# 2. Events -

| ID | Event | Source |
|---|---|---|
| P1 | <b>assigned_spot_reads_tag</b> | The assigned spot's RC522 |
| P2 | <b>other_spot_reads_tag | A different spot's RC522 |
| P3 | <b>assigned_spot_reads_foreign_tag</b> | The assigned spot's RC522, different UID |
| P4 | <b>travel_timeout</b> | Pi timer, vehicle not yet reported |
| P5 | <b>vehicle_stopped_in_lane</b> | Lane sensing / no spot report within window |
| P6 | <b>vehicle_moved</b> | Lane sensing | 
| P7 | <b>breakdown_timer_expired</b> | Pi timer, 2 min |
| P8 | <b>spot_became_free</b> | Spot mschine |
| P9 | <b>no_spot_free</b> | Spot machine </b> |
| P10 | <b>camera_granted</b> | Camera Stage Machine |
| P11 | <b>camera_wait_timeout</b> | Pi timer |
| P12 | <b>camera_unavailable</b> | Camera stage is <b>DISABLED</b> |
| P13 | <b>alignment_ok</b> | Vision result |
| P14 | <b>alignment_off</b> | Vision result |
| P15 | <b>attempts_ezhausted</b> | Attempt counter reached 5 |
| P16 | <b>vehicle_absent</b> | Staff-wait timer expired, spot reports no tag |
| P17 | <b>vehicle_present</b> | Staff-wait timer expired, spot still reports tag |
| P18 | <b>grace_timer_expired</b> | Pi timer |

# 3. Transition Table -

| No. | Current | Event | Guard | Action | Next |
|---|---|---|---|---|---|
| P-T1 | --- | handed from entrance | --- | status <b>PROCESSING</b> | <b>IN_TRANSIT</b> |
| P-T2 | <b>IN_TRANSIT</b> | P1 | --- | Spot <b>RESERVED -> HELD</b>, <b>Attempt = 0</b> | <b>AWAITING_CAMERA</b> |
| P-T3 | <b>IN_TRANSIT</b> | P2 | --- | That spot's LED red; screen <b>not Available - Go To Spot n</b>; start grace timer | <b>WRONG_SPOT</b> |
| P-T4 | <b>IN_TRANSIT</b> | P3 | --- | --- | <b>RE_ROUTING</b> |
| P-T5 | <b>IN_TRANSIT</b> | P4 | <b>Lap < limit</b> | <b>Lap + 1</b>; restart travel timer | <b>IN_TRANSIT</b> |
| P-T6 | <b>IN_TRANSIT</b> | P5 | --- | start 2-min breakdown timer | <b>STALLED_IN_LAN?E</b> |
| P-T7 | <b>STALLED_IN_LANE</b> | P6 | --- | Cancel timer | </b>IN_TRANSIT</b> |
| P-T8 | <b>STALLED_IN_LANE</b> | P7 | --- | Notify service; spot -> <b>FREE</b>; status <b>MAINTENANCE</b> | <b>MAINTENANCE</b> |
| P-T9 | <b>WRONG_SPOT</b> | P1 | --- | Wrong spot's LED off; assigned spot <b>RESERVED -> HELD</b>; <b>Attempt = 0</b> | <b>AWAITING_CAMERA</b> |
| P-T10 | <b>WRONG_SPOT</b> | P18 | --- | Escalate to service | <b>MAINTENANCE</b> |
| P-T11 | <b>RE_ROUTING</b> | P8 | --- | <b>assigned_spot = n'</b>; spot n' <b>FREE -> RESERVED</b>; screens updated | <b>IN_TRANSIT</b> |
| P-T12 | <b>RE_ROUTING</b> | P9 | --- | Hold in lane; staff notified; status <b>WAITING_FOR_SPOT</b> | <b>WAITING_FOR_SPOT</b> |
| P-T13 | <b>WAITING_FOR_SPOT</b> | P8 | --- | <b>assigned_spot = n'</b>; spot n' <b>FREE -> RESERVED</b> | <b>IN_TRANSIT</b> |
| P-T14 | <b>AWAITING_CAMERA</b> | P10 | --- | Camera moves to spot n | <b>CHECKING_ALIGNMENT</b> |
| P-T15 | <b>AWAITING_CAMERA</b> | P11 | --- | Release request; spot <b>HELD -> OCCUPIED</b>; status <b>ALIGNMENT_UNKNOWN</b> | <b>ALIGNMENT_UNKNOWN</b> |
| P-T16 | <b>AWAITING_CAMERA</b> | P12 | --- | same as P-T15 | <b>ALIGNMENT_UNKNOWN</b> |
| P-T17 | <b>CHECKING_ALIGNMENT</b> | P13 | --- | LED green; release camera; spot <b>HELD -> OCCUPIED</b>; status <b>PARKED</b> | <b>PARKED</b> |
| P-T18 | <b>CHECKING_ALIGNMENT</b> | P14 | Attempt < 5 | Buzzer every 2 s; Attempt + 1 | <b>REALIGNING</b> |
| P-T19 | <b>REALIGNING</b> | recheck due | --- | Camera re-evaluates - no move, camera still held | <b>CHECKING_ALIGNMENT</b> |
| P-T20 | <b>CHECKING_ALIGNMENT</b> | P15 | --- | Staff notified; screen <b>Please wait</b>; release camera; 1min timer | <b>AWAITING_STAFF</b> |
| P-T21 | <b>AWAITING_STAFF</b> | P16 | --- | Spot -> <b>FREE</b>; status <b>ABANDONED</b> | <b>ABANDONED</b> |
| P-T22 | <b>AWAITING_STAFF</b> | P17 | --- | Spot stays <b>HELD</b>; status <b>MISALIGNED</b> | <b>MISALIGNED</b> |

## Unexpected Events -

### Three exception :

| Current | Event | Handling |
|---|---|---|
| any terminal state | any | Ignored. A terminal session accepts nothing. |
| <b>AWAITING_CAMERA</b>, <b>CHECKING_AWAITING</b>, <b>REALIGNING</b> | P2 <b>other_spot_reads_tag</b> | Ignored. The vehicle is physically at its bay; a stray read elsewhere is noise. |
| <b>WAITING_FOR_SPOT</b> | P5<b>vehicle_stopped_in_lane</b> | Ignored. Stopping in the lane is the instruction, not a fault. |

# 4. Invariants -

### 1. Exactly one spot is <b>RESERVED</b> or <b>HELD</b> for a vehicle at any movement - never two.
### 2. <b>HELD -> OCCUPIEd</b> happens only on a camera verdict or a camera-unavailable timeout. Nothing else may set <b>OCCUPIED</b>.
### 3. The camera is released on every exit from <b>CHECKING_ALIGNMENT</b>, success or failure.
### 4. <b>Attempt</b> is reset pn entry to <b>AWAITING_CAMERA</b>, outside the realign loop.
### 5. Every non-terminal state has a timeout.
### 6. A vehicle in <b>WAITING_FOR_SPOT</b> is never towed.

# 5. Failure and degraded behaviour -

| Failure | Response | Does parking continue? |
|---|---|---|
| Camera stage disabled or not homed | <b>ALIGNMENT_UNKNOWN</b>, spot <b>OCCUPIED</b> | Yes, unverified |
| Camera queue timeout | <b>ALIGNMENT_UNKNOWN</b>, spot <b>OCCUPIED</b> | Yes, unverified |
| Spot reader fault | Vehicle never reports arrival -> travel timeout -> breakdown path | No, for that spot |
| Spot screen or LED fault | Driver gets no instruction; alignment still verified | Degraded |
| Buzzer fault | Driver is not prompted to realign; attempts still count down to staff | Degraded |

# <b>Machine 2 -- Camera Stage :</b>

# 1. States -

| State | Meaning |
|---|---|
| <b>NOT_HOMED</b> | Powered, position unknown. Initial state. |
| <b>HOMING</b> | Driving toward the endstop. |
| <b>IDLE</b> | Position known, no session holds it. |
| <b>MOVING</b> | Travelling to a requested spot. |
| <b>CHECKING</b> | In position, held by one session. |
| <b>DISABLED</b> | Homing failed or a fault occurred. Alignment unavailable system-wide. |

# 2. Transitions -

| No. | Current | Event | Action | Next |
|---|---|---|---|---|
| C-T1 | <b>NOT_HOMED</b> | system start | Drive toward endstop | <b>HOMING</b> |
| C-T2 | <b>HOMING</b> | endstop triggered | <b>position = 0</b> | <b>IDLE</b> |
| C-T3 | <b>HOMING</b> | homing timeout | Staff notified; alignment disabled | <b>DISABLED</b> |
| C-T4 | <b>IDLE</b> | request granted to session for spot n | Command move | <b>MOVING</b> |
| C-T5 | <b>MOVING</b> | in position | --- | <b>CHECKING</b> |
| C-T6 | <b>CHECKING</b> | session released | --- | <b>IDLE</b> |
| C-T7 | any | fault | Staff notified | <b>DISABLED</b> |
| C-T8 | <b>DISABLED</b> | staff reset | --- | <b>HOMING</b> |

# Invariants -

### 1. At most one session holds the stage. <b>CHECKING</b> implies exactly one holder.
### 2. No alignment verdict is valid unless the stage was in <b>CHECKING</b> when it was taken.
### 3. The endstop stops the motor on the Arduino, without reference to this machine. 
### 4. The queue is bounded at four; it cannot overflow or starve.


# 7. Interaction with other machines -

| Source | Transition | Causes |
|---|---|---|
| Entrance Gate | <b>CLOSING -> IDLE</b> | Creates this session in <b>IN_TRANSIT</b> |
| Parking Session | P-T2, P-T9 | Spot: <b>RESERVED -> HELD</b> |
| Parking Session | P-T15, P-T16, P-T17 | Spot: <b>HELD -> OCCUPIED</b> |
| Parking Session | P-T8, P-T21 | Spot: -> <b>FREE</b> |
| Parking Session | P-T11, P-T13 | Spot n': <b>FREE -> RESERVED</b> |
| Parking Session | P-T14 <-> Camera Stage | <b>IDLE -> MOVING -> CHECKING</b> |
| Parking Session | P-T17, P-T20 | Camera Stage: <b>CHECKING -> IDLE</b> |
| Camera Stage | <b>-> DISABLED</b> | Every waiting session gets P12 |
| Spot | any <b>-> FREE</b> | Every <b>RE_ROUTING</b>/<b>WAITING_FOR_SPOT</b> session gets P8 |
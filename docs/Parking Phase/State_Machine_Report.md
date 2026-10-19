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

# Events -

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

# Transition Table -

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
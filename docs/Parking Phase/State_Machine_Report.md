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
# Exit Phase - State Machine :

## Overview -
### One machine : the Exit Gate, one instance, running on the Rasberry Pi. It owns the sequence from a vehicle arriving in the exit lane to the barrier closing behind it.

### This phase is where two other machines reach the end of their lives. The Vehicle session terminates at <b>EXIT</b>, and the spot returns to <b>FREE</b>. Both transitions hang off a single event in this machine, and which event that is turns out to be the central design decision - see invariant 2.

# 1. State -

| State | Meaning - "right now the gate is..." | Flowchart region |
|---|---|---|
| <b>IDLE</b> | Nothing at the exit. Barrier closed, LED red. | Start/default |
| <b>APPROACHING</b> | Something detected in the exit lane, not yet settledat the reader. | <b>Exit sensor Detect The Vehicle</b> |
| <b>READING_TAG</b> | Vehicle at the reader. Attempting to read a tag. | Exit reader box |
| <b>VALIDATING</b> | UID in hand. Looking up the vehicle record. | Record lookup box |
| <b>OPENING</b> | Record valid. Barrier commanded open, LED green. | Barrier open box |
| <b>IN_GATE</b> | Beam broken - the vehicle is physically between the posts. | ---place--- |
| <b>CLOSING</b> | Beam restored. Barrier closing. | Barrier close box |
| <b>REFUSED</b> | Exit denied. Barrier shut, staff notified. | Refusal branch |


## Initial state : <b>IDLe</b>
## Terminal states : none. The gate cycles.

## Why one refusal state here, two at the entrance -

### Two things can go wrong at the exit: no tag could be read, or a tag was read but no open record. Both leave by the same door - a person has to come. So they are one state with reason feild:

| <b>refusal_reason</b> | Set when |
|---|---|
| <b>UNREADABLE</b> | Attempt limit reached with no UID |
| <b>NO_OPEN_RECORD</b> | UID read, but no vehicle record is open for it |

### The entrance has two refusal states because <b>REFUSED_FULL</b> resolves itself when a spot frees, while <b>REFUSED_UNREADABLE</b> cannot, Different doors, different states. Here both doors are the same door.

# 2. Events -

| ID | Event | Source |
|---|---|---|
| X1 | <b>vehicle_detected</b> | Exit lane sensing |
| X2 | <b>vehicle_cleared_lane</b> | Exit lane sensing |
| X3 | <b>tag_read(uid) | Exit RC522 |
| X4 | <b>read_timeout</b> | No tag within T |
| X5 | <b>attempt_exhausted</b> | Attempt counter reached limit |
| X6 | <b>record_valid</b> | Open record found for this UID |
| X7 | <b>record_invalid</b> | No record, or record already <b>EXIT</b> |
| X8 | <b>beam_broken</b> | Clearance sensor |
| X9 | <b>beam_restored</b> | Clearance sensor |
| X10 | <b>open_timeout</b> | Barrier open, no <b>beam_broken</b> within T |
| X11 | <b>barrier_closed</b> | Servo reached closed position |
| X12 | <b>staff_clear</b> | Staff reset |

# 3. Transition Table -

| No. | Current | Event | Guard | Action | Next |
|---|---|---|---|---|---|
| X-T1 | <b>IDLE</b> | X1 | --- | Start approach timer | <b>APPROACHING</b> |
| X-T2 | <b>APPROACHING</b> | X2 | --- | Cancel timer | <b>IDLE</b> |
| X-T3 | <b>APPROACHING</b> | vehicle settled | --- | <b>Attempts = 0</b>; screen <b>Present Tag</b> | <b>READING_TAG</b> |
| X-T4 | <b>READING_TAG</b> | X4 | <b>Attempts < limit</b> | <b>Attempts + 1</b>; screen <b>Try Again</b> | <b>READING_TAG</b> |
| X-T5 | <b>READING_TAG</b> | X3 | --- | Hold UID | <b>VALIDATING</b> |
| X-T6 | <b>READING_TAG</b> | X5 | --- | <b>refusal_reason = UNREADABLE</b>; staff notified | <b>REFUSED</b> |
| X-T7 | <b>READING_TAG</b> | X2 | --- | Abandon | <b>IDLE</b> |
| X-T8 | <b>VALIDATING</b> | X6 | --- | LED green; command open | <b>OPENING</b> |
| X-T9 | <b>VALIDATING</b> | X7 | --- | <b>refusal_reason = NO_OPEN_RECORD</b>; staff notified | <b>REFUSED</b> |
| X-T10 | <b>OPENING</b> | X8 | --- | --- | <b>IN_GATE</b> |
| X-T11 | <b>OPENING</b> | X10 | --- | Command close, Status unchanged. Spot unchanged. | <b>CLOSING</b> |
| X-T12 | <b>IN_GATE</b> | X9 | --- | Command close; status -> <b>EXIT</b>; spot -> <b>FREE</b> | <b>CLOSING</b> |
| X-T13 | <b>CLOSING</b> | X11 | --- | LED red; screen default | <b>IDLE</b> |
| X-T14 | <b>REFUSED</b> | X12 | --- | Clear reason | <b>IDLE</b> |

## Unexpected Events -

### Default : any event listed is ignored and logged.

## Three Exceptions :

| Current | Event | Handling |
|---|---|---|
| <b>OPENING</b> | X9 <b>beam_restored</b> | Ignored. Critical - see lamda5. |
| <b>OPENING</b>,<b>IN_GATE</b>,<b>CLOSING</b> | X3 <b>tag_read</b> | Ignored. A second tag presented while a car is passing must not start a new exit. |
| <b>IDLE</b> | X9 <b>beam_restored</b> | Ignored. Idle beam noise is not a departure. |

# 4. Invariants -

### 1. The barrier opens only when an open vehicle record exist for the tag presented.
### 2. status <b>EXIT</b> and the spot release happen only on <b>beam_restored</b> after <b>beam_broken</b> - never on the tag read.
### 3. The barrier never closes while the beam is broken. Enforced on the Arduino.
### 4. At most one vehicle is processed at the gate at a time.
### 5. Every state except <b>IDLE</b> has a timeout.
### 6. A vehicle that opens the gate but does not leave keeps its spot.

### Invariant 2 is the one this phase exist to guarantee. a tag read proves the car is at the gate; it does not prove the car left. Release the spot on the read and a driver who presents a tag then reverse back in has their spot given to an incoming car while they are still inside it. That is the entrance race condition arriving through the exit - same failure, different door. Invariant 6 is the same statement from the vehicle's side.

# 5. Failure and degraded behaviour -

| Failure | Response | Does the gate work? |
|---|---|---|
| Exit reader fault | <b>REFUSED</b>/<b>UNREADABLE</b>; staff open manually | No - no automatic exits |
| Beam stuck broken | No <b>beam_restored</b>; barrier stays open past timeout; staff notified | Degraded, gate held open |
| Beam stuck clear | <b>beam_restored</b> in <b>OPENING</b> is ignored; <b>open_timeout</b> fires; barrier closes; spot not released |
| Servo fault | No close confirmation; gate out of service; staff notified | No |
| Power loss | Backup battery holds the servo. On restore, barrier position unknown; command closed, then <b>IDLE</b>. | After re-init |

## Why <b>IN_GATE</b> is a separate state -

### It looks redundant - the barrier is open in both <b>OPENING</b> and <b>IN_GATE</b>, It isn't, and the stuck-clear row above is why.

### If the two were one state, a beam that is permanently reading "clear" would deliver <b>beam_restored</b> immediately after the barrier opened. The machine would mark the vehicle <b>EXIT</b>, free its spot, and close the barrier - all while the car was still sitting in front ot it. A broken sensor would silently corrupt the database.

### With <b>IN_GATE</b> in the way, beam_restored</b> is only meaningful after <b>beam_broken</b>. A stuck-clear beam can never produce that sequence, so the machine times out instead and changes nothing. The failure becomes visible rather that silent.

### That's the general principle, and it's worth stating in the report: an event that only means something in a sequence needs a state to record that the sequence started. One extra state converts a silent data-corruption bug into a timeout and a staff alert

# 7. Interaction with other machines -

| This machine | Transition | Causes |
|---|---|---|
| Exit Gate | X-T12 | Vehicle -> <b>EXIT</b> (terminal) |
| Exit Gate | X-T12 | Spot n: <b>OCCUPIED -> FREE</b> |
| Exit Gate | X-T11 | Nothing, The timeout path deliberately causes no transition elsewhere. |
| Spot -> <b>FREE</b> | broadcast | Reaches every entrance <b>REFUSED_FULL</b> and every parking <b>WAITING_FOR_SPOT</b> session |
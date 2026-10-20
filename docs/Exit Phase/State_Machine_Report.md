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
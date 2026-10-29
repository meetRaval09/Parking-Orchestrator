# Exit Phase -- Communication Protocols 

### Structurally the exit gate is the entrance gate mirrored, so almost everything is reuse -- and the reuse is itself the desugn decision. One thing is genuinely new: the single most important message in the system passes through this link.

### This phases uses two of the three links. Link 3 (camera process) is not involved -- no vision runs at the exit. Worth stating explicitly, so a reader does not go looking for it.

# 1. Reuse -- the deliberate kind

## Every verb in this phase already exists in the entrance catalogue. Only the location argument changes:

| Entrance | Exit |
|---|---|
| <b>LED,ENT,GREEN</b> | <b>LED,ENT,GREEN</b> |
| <b>LCD,ENT,<text></b> | <b>LCD,EXT,<text></b> |
| <b>SERVO,ENT,OPEN</b> | <b>SERVO,EXT,OPEN</b> |
| <b>TAG,ENT,<uid></b> | <b>TAG,EXT,<uid></b> |
| <b>BEAM,ENT,BROKEN</b> | <b>BEAM,EXT,BROKEN</b> |

### No new verbs, no exit-specific frame format, no second parser. the location field was built into the protocol from the start precisely so this phase would cost nothing to add.

### State it as a decision in the report, because a reviewer who sees two gates and one protocol will assume either good design or an oversight, and the sentence is what tells them which; the gate protocol is location-parameterised, so the entrance and exit share one firmware path and one test suite; a bug fixed at one gate is fixed at bath.

# 2. Message catalogue -- exit phase

## Pi --> Arduino

| Verb | Args | Effect |
|---|---|---|
| <b>LED</b> | <b>EXT</b>, <b>RED / GREEN / OFF</b> | Exit LED |
| <b>LCD</b> | <b>EXT</b>, <b><text></b> | Exit screen, if fitted |
| <b>SERVO</b> | <b>EXT</b>, <b>OPEN</b> | Command the barrier open |
| <b>SYNC</b> | --- | Report all device state |

### There is no <b>SERVO,EXT,CLOSE</b>. This Pi never commands the exit barrier shut. Closing is the Arduino's reflex on <b>beam_restored</b>, and a command the protocol does not contain is a command the Pi cannot issue by mistake. The ownership document claims the Arduino owns the close; this is where that claim is enforced rather than described.

## Arduino --> Pi

| Verb | Args | Meaning | State machine event |
| <b>TAG</b> | <b>EXT</b>, <b><uid></b> | Tag read at the exit reader | X3 |
| <b>NOTAG</b> | <b>EXT</b> | read window elapsed, no tag | X4 |
| <b>BEAM</b> | <b>EXT</b>, <b>BROKEN</b> | Vehicle entered the gate | X8 |
| <b>BEAM</b> | <b>EXT</b>, <b>CLEAR</b> | Vehicle cleared the gate | X9 |
| <b>SERVO</b> | <b>EXT</b>, <b>OPENED / CLOSED</b> | Barrier reached position | X11 |
| <b>FAULT</b> | <b><device></b>, <b><code></b> | Device unavailable | --- |

# 3. Why the Arduino reports two edges and not one departure 

### The Arduino sequences the beam itself -- it has to, because its close reflex fires on <b>CLEAR</b> only after <b>BROKEN</b>. So it already knows a vehicle has passed. It would be trivial to send one message:

### <b>#330, PASSED, EXT*1F</b>

### It does not, and should not. "a vehicle has deported" is a system-level fact that triggers a database write -- status <b>EXIt</b>, spot released. That is interpretation, and interpretation is the Pi's. The Arduino reports what its sensor did:

### <b>#330,BEAM,EXT,BROKEN*2A</b>
### <b>#331,BEAM,EXT,CLEAR*6D</b>

### and the Pi concludes what that means. Same rule as <b>TAG</b> in the parking phase: the Arduino transports, the Pi interprets. The protocol gives the Arduino no field in which to assert a departure, which is what makes the rule structural rather than a convention someone could quitely break while writing firmware.

### The link is a single ordered stream, so <b>BROKEN</b> cannot arrive after <b>CLEAR</b>. Only loss ia a concern, not reordering -- which matters for what comes next.

# 4. The most important message in the system

### <b>BEAM,EXT,CLEAR</b> is the message that releases a spot and closes a vehicle's record. Lose it and :

### -> the bay stays <b>OCCUPIED</b> forever
### -> the vehicle stays recorded as inside the car park
### -> the car park permanently loses one of four spot

### Every other lost message in this system produces a retry, a timeout, or a degraded record. This one produces silent, permanent corruption. It deserves more than the standard three retransmissions.

## The Second Path

### The Arduino's owns reflex gives you a free, independent confirmation. It closes the barrier only on <b>CLEAR</b> after <b>BROKEN</b>. So:

### <b>SERVO,EXT,CLOSED</b> logically implies the vehicle cleared the gate.

### Which means the Pi has two ways to learn the same fact, arriving as separate messages over the same link. Reconciliation rule:

| Pi observed | Conclusion |
|---|---|
| <b>BEAM,EXT,CLEAR</b> then <b>SERVO,EXT,CLOSED<b> | Normal, release on the beam message. |
| <b>SERVO,EXT,CLOSED</b> with no preceding <b>CLEAR</b> | The beam message was lost. Release anyway, log <b>INFERRED_FROM_CLOSE</b>. |
| <b>BEAM,EXT,CLEAR</b> with no <b>SERVO,EXT,CLOSED</b> within 5 s | Barrier fault. Release the spot, mark the gate out of service. |
| Neither, and the barrier was opened | <b>open_timeout</b> --> X-T11. Change nothing. Correct: the car never left. |

### Row two is the one that earns its place. One retransmission scheme protects againts a dropped frame; a second, independent inference path protects againts the retransmission scheme itself failing. For the one message whose loss is unrecoverable, that redundancy is proportionate.

### Row four matters just as much and says the opposite; when neither signal arrives, the system must not guess. No evidence of departure means no departure.

## Delivery settings

| Message | Retries | Interval |
|---|---|---|
| <b>BEAM,EXT,CLEAR</b> | 5 | 100 ms |
| <b>SERVO,EXT,CLOSED</b> | 5 | 100 ms |
| everything else | 3 | 100 ms |

# 5. Timing 

| Step | Time | Where |
|---|---|---|
| Beam edge --> interrupt --> close command | < 1 ms | Arduino, local |
| <b>BEAM,EXT,CLEAR</b> reaches the Pi | ~5 ms | link |
| Pi writes status <b>EXIT</b>, releases spot | ~10 ms | Pi |
| Barrier sweep | ~500 ms | servo |

### The barrier is already closing before the Pi knows the car has gone, and that is the intended order. The database catching up a few milliseconds later costs nothing; a barrier waiting on the database could cost a car's painwork.

### No two-phase commands in this phase -- nothing takes longer than teh 100 ms command timeout. The <b>MOVE</b> / <b>HOME</b> machinery from the parking phase is not needed here.

# 6. Failure modes at the protocol layer

| Failure | Symptom on the link | Response |
|---|---|---|
| Exit reader fault | <b>FAULT,RFID_EXIT,<code></b>, or <b>NOTAG</b> on every vehicle | X-T6 --> <b>REFUSED</b>, staff opens manually |
| Beam stuck broken | <b>BROKEN</b> with no following <b>CLEAR</b> | No close; timeout; gate held open, staff notified |
| Beam stuck clear | No <b>BROKEN</b> after <b>SERVO,EXT,CLOSED</b> | <b>open_timeout</b>; close; nothing released |
| Servo fault | No <b>SERVO,EXT,CLOSED</b> | Gate out of service |
| Link loss, barrier closed | Silence | Stays closed. Screen <b>Out of Service</b>. |
| Link loss, barrier open | Silence | Stays open. The Arduino's close reflex still works, so a departing car is still handled correctly and the spot is reconciled on <b>SYNC</b>. |

### That last row is the payoff of putting the close on the Arduino. with the link down, the exit gate still does the only safety-critical thing it has to do. The bookkeeping catches up when the Pi returns.

# 7. One decision -- does the exit need an approach sensor?

### Your exit state machine has <b>APPROACHING</b>, mirroring the entrance. But the two gates have different information needs :

### -> The entrance must detect an approaching car before any tag is presented, to know when to prompt <b>Present Tag</b>.
### -> The exit is used by a drive who already wants to leave. Presenting the tag is the announcement.

| Option | Consequence |
|---|---|
| A -- no approach sensor | <b>APPROACHING</b> disappears. X1, X2 vanish. <b>IDLE -> READING_TAG</b> becomes continuous polling of the exit reader. Two fewer states, one fewer sensor, two fewer pins. |
| B -- ultrasonic mirroring the entrance | Symmetry across both gates; the protocol and firmware stay identical. Costs a sensor and two pins for information you don't act on. |
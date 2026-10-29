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
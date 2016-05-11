Generic Clock
----

```
--------
```

|     Item       |   Value    |   Comment
| -------------: | ---------- | ----------------
|    Vendor code | 0x1c6c8b36 | NYA Elektriska
|      Device ID | 0x12d1b402 | Generic Clock
|    Device type | 0x12d1     | Integrated Timer, generates HWI
|        Version | 1          | Fixed Version

Device Description
----
Generic clock that ticks, occasionally tocks. 

Interrupt Commands
----

 - **0x0000**: SET_SPEED
 	- B: Clock is set to 60/B per second. Set B to zero to disable.
 - **0x0001**: GET_TICKS
 	- C: Number of ticks since last call in C
 - **0x0002**: SET_INT
 	- B: Interrupt message set to B. Zero to disable.


Behaviours
----
When interrupts are enabled, the clock will trigger an interrupt on each tick 
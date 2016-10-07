Generic Clock
----

|     Item       |   Value    |   Comment
| -------------: | ---------- | ----------------
|    Vendor code | 0x1c6c8b36 | NYA Elektriska
|  Compatible ID | 0x12d0b402 | Generic Clock (Older models)
| Device type ID | 0x12d1b402 | Generic Timer
|        Version | 2          | Fixed Version

Device Description
----

Generic clock that can be used to time short periods, and can report the
absolute time and date.

This device is backwards compatible with the original version 1 clock.


Interrupt Commands
----

 - **0x0000**: `SET_SPEED`
 	- B: Clock is set to 60/B per second. Set B to zero to disable.
 - **0x0001**: `GET_TICKS`
 	- C: Number of ticks since last call in C
 - **0x0002**: `SET_INT`
 	- B: Interrupt message set to B. Zero to disable.
 - **0x0010**: `REAL_TIME`
	- Returns the absolute time in the Common Era, in the format below.
 - **0x0011**: `RUN_TIME`
	- Returns the elapsed time since power was applied, in the format below.
 - **0x0012**: `SET_REAL_TIME`
	- Sets the absolute time, based on the format below.
 - **0xffff**: `RESET`
	- Resets to initial state, as though the DCPU was restarted. (Timer
          disabled, elapsed run time reset to 0.)


Time Format
----
All interrupts return the real time in the same format:

- `B`: Year (Earth year, CE/AD)
- `C`: Month and date
    - Month is high byte; 1 = January, 12 = December
    - Date is low byte; days into the month, 1-31.
- `X`: Hours and Minutes
    - Hours are high byte; 0-23, where 0 is midnight, 12 is noon.
    - Minutes are low byte; 0-59
- `Y`: Seconds; 0-59
- `Z`: Milliseconds; 0-999

Frame of Reference
----
The time returned by a brand new clock device is undefined. The clock must be
set from an outside source.

Implementors are encouraged to include a clock battery, to hold the time even
when power is disconnected. Consult your system's documentation.

Behaviours
----
When interrupts are enabled, the clock will trigger an interrupt on each tick.


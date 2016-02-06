Integrated Activity Control Module
----

|     Item       |   Value    |   Comment
| -------------: | ---------- | ----------------
|    Vendor code |            | (Various)
|      Device ID | 0x11E0DACC | IACM
|    Device type | 0x11E0     | Integrated CPU controller
|        Version | 0x0004     | Version/highest mode supported

Device Description
----

The integrated activity control module (or *IACM*) allows software to control on the processor: clock rate, interrupts, and power consumption.
The IACM has 2 control buttons called "Power" and "Mode"; they are physically distinct and may be labelled various ways depending on the chassis.

The IACM can be in one of several different modes determining the effects on the CPU:
 - Mode 0 - CPU runs at full rate/power as normal.
 - Mode 1 - CPU runs at reduced rate/power as normal.
 - Mode 2 - CPU runs at reduced rate/power, and is periodically put to sleep.
 - Mode 3 - CPU is put to sleep, interrupts to the CPU will cause the IACM to switch to mode 1.
 - Mode 4 - Power off the CPU, memory, and any internal hardware.

Interrupt Commands
----

when interrupting the device, register A will act as a command index as follows:

 - **0x0000**: Set Mode - Change mode to the mode number in B, unsupported modes may cause undefined behaviour.
 - **0x0001**: Set RunTime - Set the amount of time to run while in mode 2. Register B holds the amount of time in 100 millisecond increments. Setting to zero resets to default.
 - **0x0002**: Set SleepTime - Set the amount of time to sleep while in mode 2. Register B holds the amount of time in 100 millisecond increments. Setting to zero resets to default.
 - **0x0003**: Set interrupt message - Sets the message to the value in B for the interrupt type in C.
   - Type 0: If message is non-zero, generate an interrupt when the "Power" button is pressed.
   - Type 1: If message is non-zero, generate an interrupt when the "Mode" button is pressed.
 - **0x0004**: Get clock rate - gets the normal CPU clock rate in Hz as a 32 bit number, B holds the lower 16 bits, C holds the upper 16 bits.
 - **0x0005**: Get r-clock rate - gets the reduced CPU clock rate Hz as a 32 bit number, B holds the lower 16 bits, C holds the upper 16 bits.
 - **0x0505**: Force the system to reset everything (including the IACM)

Behaviour Details
----

When power is applied to the system or if the CPU resets, the IACM defaults to mode 0, powering up the CPU and running at full rate.
While in mode 4, pressing the "Power" button will switch to mode 0, powering up the CPU.
Holding the "Power" button for 4 seconds while in any mode other than 4, will force the IACM to enter mode 4 and power off the CPU, memory, etc.

The reduced clock rate for CPUs is CPU specific, but usually 10th the normal rate.

#### Mode 2
The CPU will run for the amount of time set by "RunTime", then be put to sleep for the amount of time set by "SleepTime".
If the CPU gets any interrupts from hardware (including the IACM), then it will be switched to running.
The transition between run and sleep is controlled by a free running timer in the IACM; if the CPU switches to running because an interrupt during SleepTime, the CPU will run for the remainder of SleepTime plus the following RunTime.
The default for RunTime and SleepTime is 5 (500 milliseconds), and is set at power on or CPU reset.


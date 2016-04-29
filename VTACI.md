Vectored Truster Array Control Interface
-----

```
  \ \\ Rin Yu Research Group
  /\ /    凜羽研究小組

```

|     Item       |   Value    |   Comment
| -------------: | ---------- | ----------------
|    Vendor code | 0xC2200311 | Rin Yu Research
|      Device ID | 0xF7F7EE03 | VTACI
|    Device type | 0xF7F7     | Trust Control Card
|        Version | 0x0400     | Version 4.0

VTACI is advanced control of vector impulse trusters, many thruster are support
on each device with full or automatic control of thruster in groups.

VTACI have two control mode for operation of thrusters
 - Direct Group Mode - Software on DCPU control each thruster.
 - Vector Process Mode - Software on DCPU sends moment/force and VTACI controls.

New installs of VTACI and thruster are calibrated, if add more thrusters, VTACI
must be re-run calibration process for proper function. Calibrate is require
for correct operation of VTACI in any mode, recommended always calibrate device.

Interrupt Commands
----

DCPU HWI Interrupts VTACI and makes use of register A in DCPU to select action:

 - **0x0000**: Stop All Thrust

   Disable all thruster and turn off all memory map control.

 - **0x0001**: Calibrate Thrusters

   Important to do when installed, takes 10 seconds plus 2 for each thruster.
   Only need to do one time or when thrusters added.

 - **0x0002**: Calibrate Status

   Will set DCPU register A to value for status:
   - 0 - device is OK and Calibrate
   - 1 - device is busy run calibrate process
   - 2 - device is need calibrate run
   - 0xffff - device is damage or crashed, need to power off and on to fix.

   Will set DCPU register B to number of attach thrusters.
   Will set DCPU register C to number of attach gimbles.

 - **0x0003**: Set Thrust mode

   DCPU register C value is use to pick mode:
   - 0 - disable mode, turn off all thruster
   - 1 - force and moment mode, use memory at address from DCPU register B
   - 2 - group control mode, use memory at address from DCPU register B
   - other values are not correct and VTACI will do nothing different.
   - more detail is later, B is *base address*.

 - **0x0004**: Get Information of Thrusters

   store information at memory at address from DCPU register B.
   5 memory words are need for each thruster and is stored information:
   - N+0 - X vector in mm from center of mass or VTACI.
   - N+1 - Y vector in mm from center of mass or VTACI.
   - N+2 - Z vector in mm from center of mass or VTACI.
   - N+3 - Hex number of direction, 0xWZYX, 0 = -1, 8 = 0, F = +1;
     W = 0: from center of mass, W = 1: from center of VTACI.
   - N+4 - Hex number of thrust rated, 0xETTT, Thrust = (TTT x 10 ^ ( -6 + E ))
     newtons.

   DCPU register A will get set to number of thrust information.
   Memory use is A x 5.
   DCPU register C will set to 0 if OK, or set to 1 if not calibrate.
   No memory will set if C is not 0 and OK, and A will set to 0.

 - **0x0005**: Get Information of Gimbles

   store information at memory at address from DCPU register B.
   2 memory words are need for each gimble and is stored information:
   - N+0 - Hex number 0xYYXX, XX and YY are range of motion 00=0, FF=90 degree.
   - N+1 - Number of thruster attached to gimble.

 - **0x0006**: Set Truster Groups

   read information at memory at address from DCPU register B.
   1 memory word are need for each thruster, offset (group) for thruster.
   This only need for thrust group mode.
   Each thruster will use 1 word at offset + base for control.

   Largest offset number is amount of memory read for control.

   **IMPORTANT** *Larger max offset number reduce response time.*

 - **0x0007**: Set Gimble Groups

   read information at memory at address from DCPU register B.
   1 memory word are need for each gimble: offset (group*2) for gimble.
   Each gimble will use 2 words at offset + base for control.

   Largest offset number + 1 is amount of memory read for control.

   **IMPORTANT** *Larger max offset number reduce response time.*

 - **0x0008**: Thrusters Status

   write information at memory at address from DCPU register B.
   1 memory word are need for each thruster, the status of thruster.

    - 0 - Thruster off and OK.
    - 1 - Thruster firing and OK.
    - 0x100 - Thruster not respond to VTACI command.
    - 0x101 - Thruster overheat.
    - 0x102 - Thruster on malfunction gimble.

   **IMPORTANT** *Use of 10+N DCPU cycle for interrupt, N=number thrusters*

   Will set DCPU register A equal number of error thrusters.

 - **0xffff**: Reset

   Reset offset groups, disable all thrusters, disable mode.
   Will cancel in progress calibrate, but will not delete already calibrate data.

Behaviours
----

VTACI have 3 mode of operation:
 - Disable mode - thrusters off
 - Group Control - memory control of individual thruster or group of thruster
 - Moment+Force Mode - memory control of automatic group of thruster

**IMPORTANT** *VTACI must run calibrate before first use or if thruster added.*

**IMPORTANT** *Thruster should not be attach more than 32m from center of mass*
*for Moment+Force mode. or 32m from VTACI in total.*

VTACI will calibrate for thrusters offset from center of mass if possible,
otherwise calibrate for thrusters offset from center of VTACI.

##### Moment+Force Mode

VTACI will automatically compute what thrusters should fire for moment or force.
Output rate for force and moment are added and clamped to maximum.

VTACI will read 6 words of memory at *base address* for this mode:
 - B+0: X vector of force
 - B+1: Y vector of force
 - B+2: Z vector of force
 - B+3: X vector of moment
 - B+4: Y vector of moment
 - B+5: Z vector of moment

For force: X+ is "right", Y+ is "forward", Z+ is "up".
For moment: vector+ is clockwise towards axis+.

Each force vector will fire all thrusters with opposite axis direction.

Each moment vector will fire all thrusters on all side of axis that will contribute
moment force.

As example: Y+ moment would apply to:
 - X+ thrusters below Z plane
 - X- thrusters above Z plane
 - Z+ thrusters right X plane
 - Z- thrusters left X plane

All vectors can scale thrust output levels from 0% to 100%.
Force+Moment mode will center all gimbles and not use them.


##### Group Control Mode

Mode for larger craft using VTACI, gimble control, or just more direct control.

VTACI will read *max offset* memory words starting at *base address*.
Each thruster will read 1 word at assigned offset, that is scaler output level.
0 = Off, 65535 = Full On. all thruster with same offset get same value.
Each gimble will read 2 words, that is vector direction, X then Y.
Numbers -32767 is full axis minus, 0 is center, 32767 is axis plus.
Number -32768 is also full axis minus.
All gimble with same offset get same vector direction.

**IMPORTANT** *base address is use for both gimble and thrusters. Offset*
*of gimble and thruster overlapping can be undesired.*

**IMPORTANT** *not setting offsets before using this mode can be undesired.*

**IMPORTANT** *Larger max offset will reduce respone time.*

-----

Distributed for VTACI devices by Rin Yu Research Group.

**WARNING** *Use of device for any non-research activity may void any warrenty.*

```
  "Knowledge helps you make a living; wisdom helps you make a life."
     -- Director Rin Yu.
```


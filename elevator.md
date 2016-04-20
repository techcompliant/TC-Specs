
Lift Control Module
----

**WARNING**
*Improper use and servicing of lift equipment may cause severe injury,*
*electric shock, or death. To be serviced by licensed lift technicians only.*

#### Overview

The lift control module is an automated system to control passenger elevation
equipment. The LCM is designed as a drop-in control for stand-alone unattended
automated lifts, while also providing features for more advanced external control.

#### Basic operation

The lift control provides an easy to use external interface to allow passengers
to safely control the device. The interface consists of the button control
panel in the lift car, and one or two *call* buttons outside each lift stop
(floor) for up/down as applicable.

Pressing either of the *call* buttons at a lift stop will send a request to
the lift car to travel to that floor.

Pressing a *floor select* button on the interface inside the lift car will
request it travel to the corresponding floor.

When idle, any request will cause the lift to immediately begin travelling to
the requested floor. When the lift reaches any requested floor (or is already
at the requested floor), it will stop; open the access doors; attempt to close
the access doors after a delay; and once closed, either travel to the next
requested floor or return to idle state.

All lift requests are by default processed according to this priority:

1. Any lift stop in the travel path currently internally requested or calling
   in the same travel direction.
2. A first-in first-out (FIFO) queue of floors requested by the internal panel.
3. A first-in first-out (FIFO) queue of external lift stop requests
   (that have not been visited yet).

##### Example
 - Lift is at F1
 - F3 calls lift to go up (3)
 - Lift begins travel (up)
 - Lift stops at F3
 - F7 then F5 requested by internal panel (2)
 - Lift begins travel (up)
 - F4 calls lift to go up (3)
 - F6 calls lift to go down (3)
 - Lift stops at F4 (1)
 - Lift stops at F5 (1)
 - Lift stops at F7 (2)
 - Lift begins travel (down)
 - Lift stops at F6 (3)
 - F2 is requested internally (2)
 - Lift stops at F2 (2)
 - Lift returns to idle.

#### Override

The LCM can be put in *override* mode, using a key or override command.
In this mode, the LCM will ignore/cancel all lift calls and requests, the lift
car may move up and down and stop freely (at points other than lift stops),
and the doors may open and close freely.

**WARNING**
*Having the lift travel with the doors open will void the installation*
*warranty and may cause damage to the lift car, internal/external doors.*
*Improper operation in override mode may cause injury or death.*

Diagnostic Interface
----

The Lift control module features an external signalling interface to provide
advanced control. Commands can be sent over this interface to control the lift,
queue, doors, and a number of other things.

##### Control Commands
| Sequence      | Name        | Description                           |
| ------------- | ----------- | ------------------------------------- |
| 0x0000 0x00nn | CALL_EXT    | Call lift to floor *nn* using external FIFO (both directions) |
| 0x0001 0x00nn | CALL_INT    | Request lift to floor *nn* using internal FIFO |
| 0x0002 0x00nn | CALL_PRI    | Send lift to directly to floor *nn* with out stopping at other floors |
| 0x0003 0x00nn | SET_ACC_R   | Set floor *nn* to *restricted* access, ignore all internal requests for it |
| 0x0004        | LIFT_STOP   | Stop at next floor (if travelling), chime, and/or open door |
| 0x0005        | DOOR_CLOSE  | Safely close the lift door, if open.  |
| 0x0006        | CHIME_1     | Sound the arrival chime.  |
| 0x0007        | CHIME_2     | Sound the progress chime. |
| 0x0008        | CHIME_3     | Sound the alert chime.    |
| 0x0009 0x00nn | DISPLAY     | Change the current floor status display to *nn* |
| 0x000A 0x00nn | CALL_CNCL   | Remove all calls/requests for *nn* from FIFOs   |
| 0x000B 0x000N | CAR_CNTL    | Bits turn devices on/off (see Device control)  |
| 0x000C        | SET_EXT     | Put the LCM in external control mode, in this mode: requests/calls will not be added to FIFOs unless no command is received in 30 seconds. |
| 0x000D        | DIAG        | LCM sends 0xADXN - X is status bits (see below), N is devices status |
| 0x000E        | ECHO        | LCM sends 0xA00E  |
| 0x000F        | MODE_RESET  | Return to default mode, external mode is disabled, performs **COVERRIDE** if override mode was active | 
| 0x0010        | GET_LSTOP   | LCM sends 0xAAnn where *nn* is the current floor or last floor passed while travelling |
| 0x0011 0x00nn | GET_ACC_R   | LCM sends 0xA001 if *nn* is *restricted* access or 0xA000 if not. |
| 0x0020        | OVERRIDE    | Put LCM in override mode. |
| 0x0021 0x000n | TRAVEL_UP   | Begin moving up at speed *n*, 0 means stop   |
| 0x0022 0x000n | TRAVEL_DN   | Begin moving down at speed *n*, 0 means stop |
| 0x0023        | DOOR_OPEN   | Force doors to open. |
| 0x0024        | FDOOR_CLOSE | Force doors to close **without safety checks** (use DOOR_CLOSE instead where possible) |
| 0x002F        | COVERRIDE   | Disable override mode, safely close doors and travel to nearest lift stop. |

##### Progress Messages
The LCM will send progress messages while in external control mode.

| Action            | Sent Sequence | Detail                                     |
| ----------------- | ------------- | ------------------------------------------ |
| Generic No        | 0xA000        | Sent by any GET_* for simple Yes/No status |
| Generic Yes       | 0xA001        | Sent by any GET_* for simple Yes/No status |
| Lift Travelling   | 0xA002        | Lift has begun moving                      |
| Generic Acknowledge | 0xA003      | Sent by any command that does not send other status |
| Lift Parking      | 0xA004        | Sent before Lift parks and opens door      |
| Door Closed       | 0xA005        | Door safely closed successfully            |
| Request Timeout   | 0xA006        | A command was not sent in the external control mode timeout, and the default action is being taken. |
| Echo Response     | 0xA00E        | Sent in response to ECHO command           |
| Door Forced Open  | 0xA023        | Door forced open by command or physical action |
| Lift Stop Passed/Reached | 0xAANN | NN=Floor                                   |
| Diagnostic        | 0xADXN        | Sent in response to DIAG command, X/N are status bit fields. |
| External Call     | 0xCLNN        | L=direction up(1)/down(0) NN=Calling floor |
| Internal Request  | 0xC2NN        | NN=Requested floor                         |

##### System status bits
 - Bit 0 - Stopped (0) / Travelling (1)
 - Bit 1 - Door Closed (0) / Door Open (1)
 - Bit 2 - Last travel direction: Down (0) / Up (1)
 - Bit 3 - Queues Empty (0) / Busy (1)

##### Device Control/Status Bits
Setting to 1 enables a device, a status of 1 means enabled AND *working*.
Setting to 0 disables, a status of 0 means disabled OR broken.
 - Bit 0: Main lights
 - Bit 1: Emergency lights
 - Bit 2: Vent fan
 - Bit 3: Chime


KaiComm Remote Activity Control Module
----

```
		 |     \ Kai 
		>|     \\ Communications
		>|     \\\
		>|     ///
		>|    //
		 |   /
```

|     Item       |   Value    |   Comment
| -------------: | ---------- | ----------------
|    Vendor code | 0xA87C900E | (KaiComm)
|    Device type | 0x02       | (Communications)
| Device Subtype | 0x05       | (Remote Control)
|      Device ID | 0x30       | 
| DCPU Device ID | 0xD0F090C0 | (KaiComm RACM)
|        Version | 0x0001     | 

The KaiComm RACM is a Remote Activity Control Module designed for satellites and other unattended DCPU-based systems. 
The device monitors the state of the system by means of a configurable watchdog timer. If the DCPU does not periodically reset the timer, or upon wireless recipt of a configurable PIN, the device enters safe mode. While in safe mode, a remote reset command will reboot the attached DCPU system. While in safe mode, upon recipt of a rescue program, the device will reset the DCPU and execute the program at address 0x0000.

**Note** - Radiofrequency communication can be interfered with by physical obstructions, solar flares, and other radio transmissions. Datagrams received may have been truncated, partially replaced, or otherwise corrupted in transit.

The integrated radio covers the band from 902 MHz to 927.6 MHz, which is divided into 256 channels of 100 KHz bandwidth.

The default channel is 0x0000, or 902.0 MHz.

The default safe mode PIN is: 0x1234

The default watchdog timer value is: 0xffff

Commands
----

On interrupt, register A holds a command that the RACM will perform:

 - **0x0000**: Query status:
   Sets Register A with watchdog timer value, in number of milliseconds until safe mode is activated. A value of 0 indicates that safe mode is currently active, and that the device is accepting reset and upload commands.
   Sets register B with safe mode PIN.
 - **0x0001**: Reset watchdog timer value:
   Register B gives the number of milliseconds until the watchdog timer should trigger and put the device into safe mode. If the device is currently in safe mode, resetting the watchdog timer will leave safe mode.
 - **0x0002**: Configure PIN:
   Register B selects new safe mode PIN.
 - **0x0003**: Configure radio:
   Register B holds the channel to tune to, from 0x0000 through 0x00FF.
   Register C will be set to 0 if the settings are accepted an applied, and 1 if the proposed settings are invalid.

Remote Commands
----

The RACM accepts commands wirelessly from compatible wireless transmitters. All commands have the two-word KaiComm command header, 0xA87C, 0x900E. These commands are structured as follows:

- **PIN Command**: 0xA87C, 0x900E, 0x0001, <1-word PIN>
  Submits a PIN code to the device. If the PIN code matches the configured safe mode PIN, the device will enter safe mode.
- **Reset Command**: 0xA87C, 0x900E, 0x0002
  Reboots the attached DCPU system. If the device is in safe mode, the DCPU and all attached hardware will be reset to their default states, memory will be cleared, and the system boot process will proceed as normal.
- **Upload Command**: 0xA87C, 0x900E, 0x0003, <128-word rescue program>
  Uploads a rescue program. If the device is in safe mode, the DCPU will be stopped, the uploaded 128-word program will be coppied to memory address 0 (leaving all other memory contents unchanged), the DCPU registers and all attached hardware will be reset to their default configurations, and execution will resume from address 0. **Note** - Wireless communication can be unreliable! There is some probability that the rescue program will be truncated or contain errors. KaiComm is not responsible for any damage that may result from the use of this feature.

----


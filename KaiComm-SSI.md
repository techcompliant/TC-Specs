KaiComm Synchronous Serial Interface
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
| Device Subtype | 0x03       | (Sync serial)
|      Device ID | 0x20       | (SSI)
| DCPU Device ID | 0xE57D9027 | (KaiComm SSI 2)
|        Version | 0x0103     |

The KaiComm SSI is a bi-directional data port.
Transmissions in either direction are independent of each other and can operate at different bauds.
The size of the data primitive is selectable ranging from 1 to 4 octets.
All data are assumed to be sent MSB first.

Commands
----

On interrupt, register A holds a command that the SPI will perform:

 - **0x0000**: Query status
   Sets Register A with status, bits will be set/clear as follows:
   - Bit 0 - Line is busy/not-connected (1), or idle (0)
   - Bit 1 - Receiving Active (1), or idle (0)
   - Bit 2 - Data available (1), or not (0)
   - Bit 3 - Transmitting Active (1), or idle (0)
   - Bit 4 - Transmit buffer is free (1), or full (0)
   - Bit 5-13 Reserved always set to 0
   - Bit 14 - Receive interrupts enabled: yes (1), no (0)
   - Bit 15 - Transmit done interrupts enabled: yes(1), no (0)
 - **0x0001**: Configure port
   Register B holds the octet size minus 1 in bits 0-1, other bits are ignored.
   Register C holds the baud selection in bits 0-7, giving a range of 0-255.
   The baud is calculated like so: baud = 3125 * (b+1)
   Possible bauds range from 3125 to 800,000 bits per second, in steps of 3125 bits per second.
 - **0x0002**: Receive data
   Register A is set to the lower 16 bits, register B is set to the upper 16 bits.
   Register C is set with the error status, receive error status is cleared when read.
   when configured for less than 4 octets unused bits are set to 0.
   Receive Error status codes:
   - 0x0000 - No error
   - 0x0001 - Receive buffer overflow
   - 0x0002 - Transmission was not a multiple of 8 bits
   - 0x0003 - No data available
 - **0x0003**: Transmit data
   Register B contains the lower 16 bits, register C contains the upper 16 bits.
   when configured for less than 4 octets unused bits are ignored.
   Register C is set with error status.
   Transmit Error status codes:
   - 0x0000 - No error
   - 0x0001 - Transmit buffer overflow (data will not be sent)
   - 0x0002 - Line is busy or not connected (data will be queued)
 - **0x0004**: Configure interrupts
   Register B defines which interrupts to enable.
   - Bit 0 - Interrupt on Receive enabled: yes (1), no (0)
   - Bit 1 - Interrupt on Transmit done enabled: yes(1), no (0)
   - Bit 2-15 Ignored.
   Register C contains message for Interrupt on Receive.
   Register X contains message for Interrupt on Transmit.

----

2014 Meisaka Yukara (CC-BY-SA 4.0)

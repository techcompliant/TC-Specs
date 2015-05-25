KaiComm Synchronous Parallel Interface
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
| Device Subtype | 0x00       | (Parallel port)
|      Device ID | 0x10       | (SPI)
| DCPU Device ID | 0xE0239088 | (KaiComm SPI 2)
|        Version | 0x0100     |

The KaiComm SPI is a bi-directional multipurpose data port.
Transmissions in either direction are independent of each other and can operate asynchronously of one another.
The size of the data primitive is selectable and can be either 2 or 4 octets.
The device sends "upper" octets first, if a device set to 2 receives from a device set to 4 the "upper" octets will be received, followed by the "lower" octets.

Commands
----

 - **0x0000**: Query status
   Sets Register A with status, bits will be set/clear as follows:
   - Bit 0 - Line is busy/not-connected (1), or idle (0)
   - Bit 1 - Always reads as 0
   - Bit 2 - Data available (1), or not (0)
   - Bit 3 - Always reads as 0
   - Bit 4 - Transmit buffer is free (1), or full (0)
   - Bit 5-13 Reserved always set to 0
   - Bit 14 - Receive interrupts: enabled (1), disabled (0)
   - Bit 15 - Transmit complete interrupts: enabled (1), disabled (0)
 - **0x0001**: Configure port
   Register A selects data size, 16 bit (0) or 32 bit (1), other bits are ignored.
 - **0x0002**: Receive data
   Register A is set to the lower 16 bits, register B is set to the upper 16 bits.
   Register C is set with the error status, receive error status is cleared when read.
   When configured for 2 octets, register B to 0.
   Receive Error status codes:
   - 0x0000 - No error
   - 0x0001 - Receive buffer overflow
   - 0x0002 - Receive error
   - 0x0003 - No data available
 - **0x0003**: Transmit data
   Register A contains the lower 16 bits, register B contains the upper 16 bits.
   when configured for less than 4 octets unused bits are ignored.
   Register C is set with error status.
   Transmit Error status codes:
   - 0x0000 - No error
   - 0x0001 - Transmit buffer overflow (data will not be sent)
   - 0x0002 - Line is busy or not connected (data will be queued)
 - **0x0004**: Configure interrupts
   Register A defines which interrupts to enable.
   - Bit 0 - Interrupt on Receive: enabled (1), no (0)
   - Bit 1 - Interrupt on Transmit complete: enabled (1), no (0)
   - Bit 2-15 Ignored.
   Register B contains message for Interrupt on Receive.
   Register C contains message for Interrupt on Transmit.

----

2014 Meisaka Yukara (CC-BY-SA 4.0)


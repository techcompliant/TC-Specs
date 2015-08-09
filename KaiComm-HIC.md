KaiComm Hardware Interface Card
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
|        Version | 0x02xx     | low byte is max ports - 1

The KaiComm SPI is a bi-directional multipurpose data port.
Transmissions in either direction are independent of each other and can operate asynchronously of one another.
The size of the data primitive is selectable and can be either 2 or 4 octets.
The device sends "upper" octets first, if a device set to 2 receives from a device set to 4 the "upper" octets will be received, followed by the "lower" octets.

The low byte of the version specifies the maximum number of ports available on the card.

Commands
----

On interrupt, register A holds a command that the SPI will perform:

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
   Register B selects data size, 16 bit (0) or 32 bit (1), other bits are ignored.
 - **0x0002**: Receive data
   Register A is set to the lower 16 bits, register B is set to the upper 16 bits.
   Register C is set with the error status, receive error status is cleared when read.
   When configured for 2 octets, register C to 0.
   Receive Error status codes:
   - 0x0000 - No error
   - 0x0001 - Receive buffer overflow
   - 0x0002 - Receive error
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
   - Bit 0 - Interrupt on Receive: enabled (1), no (0)
   - Bit 1 - Interrupt on Transmit complete: enabled (1), no (0)
   - Bit 2-15 Ignored.
   Register C contains message for Interrupt on Receive.
   Register X contains message for Interrupt on Transmit.
 - **0x0005**: Get number of ports
   Register A is set to number of ports.
 - **0x0006**: Get port info
   Register B selects which port number to get information about (0x0000 being the first port, 0x0001 the second etc).
   Register A is set to the port's ID number.
   Register B is set to 0x0001 if the port is currently connected, otherwise it is set to 0x0000.
   Register C is set to the error status:
   - 0x0000 - No error
   - 0x0001 - Port number out of bounds - The port number selected is too high for number of connected ports
 - **0x0007**: Get port name
   Register B selects which port number to get the name of.
   Register C passes the memory address to populate.
   The memory starting at the address held by C is populated with the name of the port, plus a null terminating character (0x0000).
   Register C is set to the error status:
   - 0x0000 - No error
   - 0x0001 - Port number out of bounds - The port number selected is too high for number of connected ports
   - 0x0002 - Insufficient memory space - The starting address + size of name + 1 exceeds 0xFFFF
 - **0x0008**: Get active port number
   Register A is set to the number of the currently active port.
 - **0x0009**: Set active port
   Register B selects which port to make the active port.
   Register C is set to the error status:
   - 0x0000 - No error
   - 0x0001 - Port number out of bounds - The port number selected is too high for number of connected ports


----

2014 Meisaka Yukara (CC-BY-SA 4.0)


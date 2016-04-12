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
| DCPU Device ID | 0xE0239088 | (KaiComm HIC)
|        Version | 0x0442     | 8 Port card
|        Version | 0x0444     | 16 Port card
|        Version | 0x0448     | 32 Port card

The KaiComm HIC (model 44) is a bi-directional multifunction data interface, with multiple ports per card.
An integrated out-of-band (OOB) signalling function provides additional information about connected devices.

Transmissions in either direction are independent of each other and can operate asynchronously of one another.
Each port contains a 2 word receive buffer, allowing it to receive on multiple ports simultaneously.
The HIC device contains a single transmitter and 2 word buffer used by all ports, the HIC can only send on a single port at a time.

The low bits of the version can be used to determine the maximum number of ports available on the card.

Commands
----

On interrupt, register A holds a command that the HIC will perform:

 - **0x0000**: Query status
   - On interrupt, register C should hold a port number to get status about.
   - Only the lowest 4 bits can be different per port, as the HIC has a single transmitter.

   - Sets Register A with info status, bits will be set or cleared as follows:
     - Bit 0 - Port is busy receiving (1), or idle/not receiving (0)
     - Bit 1 - Port is not connected (1), or connected (0)
     - Bit 2 - Data available on port (1), or not (0)
     - Bit 3 - Port number is invalid (1), or valid (0)
     - Bit 4 - Transmit buffer is free (1), or full (0)
     - Bit 5 - Transmitter is idle/empty (1), or sending (0)
     - Bit 6 - At least one port has data available (1), or none (0)
     - Bit 7-13 are reserved and always set to 0
     - Bit 14 - Receive interrupts are: enabled (1), disabled (0)
     - Bit 15 - Transmit complete interrupts are: enabled (1), disabled (0)
   - Sets Register C to lowest port which has data available (or 0xFFFF if no data is available)

 - **0x0001**: Receive data
   - On interrupt, register C should hold the port number to receive from.
   - Register B will be set to the data received, or 0 on error.
   - Register C is set with the error status, receive error status is cleared when read.

   Receive Error status codes:
   - 0x0000 - Success, No error
   - 0x0001 - Success, Receive buffer overflowed (B is still set with buffer data)
   - 0x0002 - Fail, Receive error
   - 0x0003 - Fail, No data available

 - **0x0002**: Transmit data
   - On interrupt, Register B should contain 16 bit value to send, register C should be set to the port number to send on.
   - Register C will be set with a transmit Error status code:
     - 0x0000 - No error
     - 0x0001 - Port is busy (data will be queued)
     - 0x0002 - Transmit buffer overflow (data will not be sent)
     - 0x0003 - Port is not connected (data will not be sent)
     - 0x0004 - The HIC is busy sending on another port (data will not be sent)

 - **0x0003**: Configure interrupts:
   - Register B contains message for Interrupt on Receive.
   - Register C contains message for Interrupt on Transmit.
   - A message value of 0 means disable interrupt.

 - **0x0004**: Load port name
   - On interrupt, register B should be set to the starting memory address to populate.
   - Register C should be set to the port number to get the name of.

   The memory starting at the address held in B is filled with the 8 words containing name of the port. In the case of shorter names, the additional words will be 0. A port name may not exceed 8 words.

   Register C will be set to the error status:
     - 0x0000 - No error
     - 0x0001 - Port number out of bounds - The port number selected is too high for number of built-in ports
     - 0x0002 - Invalid memory address - The starting address + 8 exceeds 0xFFFF

----

2014-2016 Meisaka Yukara (CC-BY-SA 4.0)


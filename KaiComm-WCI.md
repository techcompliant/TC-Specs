KaiComm Radiofrequency Communication Interface
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
| Device Subtype | 0x04       | (Wireless Datagram)
|      Device ID | 0x20       | (SSI)
| DCPU Device ID | 0xD00590A5 | (KaiComm RCI)
|        Version | 0x0010     |

The KaiComm RCI is a half-duplex datagram-based radiofrequency communications device.
Datagrams of up to 256 words can be transmitted, at a rate of 3,125 bits per second. Radio tuning is accomplished by external hardware.
Datagrams are framed and length-prefixed by the hardware.

**Note** - Radiofrequency communication can be interfered with by physical obstructions, solar flares, and other radio transmissions. Datagrams received may have been truncated, partially replaced, or otherwise corrupted in transit.

All datagrams are broadcast to all stations in range.

The RCI can buffer a single incoming datagram. If a new datagram is received while a datagram is currently in the buffer, the new datagram is discarded.

The RCI can buffer a single outgoing datagram. Any attempts to transmit another datagram while the buffered datagram is being transmitted will fail.

The RCI cannot transmit and receive simultaneously. If a datagram arrives while a datagram is being transmitted, the arriving datagram will be lost. Moreover, it is likely that the two datagrams will overlap and that neither will be received by any station.

Commands
----

On interrupt, register A holds a command that the WCI will perform:

 - **0x0001**: Receive Datagram:
   Register B gives the address of a 256-word buffer which will be populated with the received datagram.
   Register A will be set to 0 if a datagram was successfully retrieved from the receive buffer, and 1 if no buffered datagram was available.
 - **0x0002**: Send Datagram:
   Register B holds the address of a buffer containing the datagram to send.
   Register C holds the length of the datagram to transmit (up to 256 words).
   Register A will be set to 0 if the datagram is successfully queued for transmission, and 1 if there is already a datagram being transmitted.
 - **0x0003**: Configure interrupts
   Register B contains the interrupt message to send when a datagram is received, or 0x0000 to disable receive interrupts.
   Register C contains the interrupt message to send when a datagram transmission completes, or 0x0000 to disable transmit interrupts.


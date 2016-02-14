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
Datagrams of up to 256 words can be transmitted, at a rate of 3,125 bits per second.
Datagrams are framed and length-prefixed by the hardware.

**Note** - Radiofrequency communication can be interfered with by physical obstructions, solar flares, and other radio transmissions. Datagrams received may have been truncated, partially replaced, or otherwise corrupted in transit.

The integrated radio covers the band from 902 MHz to 927.6 MHz, which is divided into 256 channels of 100 KHz bandwidth. The radio also supports 8 transmit power levels.

The default channel is 0x0000, or 902.0 MHz, and the default transmit power level is 7.

All datagrams are broadcast to all stations in range that are tuned to the same channel.

The RCI can buffer a single incoming datagram. If a new datagram is received while a datagram is currently in the buffer, the new datagram is discarded.

The RCI can buffer a single outgoing datagram. Any attempts to transmit another datagram while the buffered datagram is being transmitted will fail.

The RCI cannot transmit and receive simultaneously. If a datagram arrives while a datagram is being transmitted, the arriving datagram will be lost. If a datagram is transmitted while a datagram is being received, the datagram being received will be discarded. Moreover, it is likely that the two datagrams will overlap and that neither will be received by any station.

Commands
----

On interrupt, register A holds a command that the WCI will perform:

 - **0x0000**: Query Status Datagram:
   Register A will be set to transmit/receive status: 0 if radio is inactive, 1 if radio is transmitting, and 2 if radio is receiving.
   Register B will be set to the current channel, from 0x0000 to 0x00FF.
   Register C will be set to the current transmit power level, from 0x0000 to 0x0007.
 - **0x0001**: Receive Datagram:
   Register B gives the address of a 256-word buffer which will be populated with the received datagram.
   Register A will be set to 0 if a datagram was successfully retrieved from the receive buffer, and 1 if no buffered datagram was available.
 - **0x0002**: Send Datagram:
   Register B holds the address of a buffer containing the datagram to send.
   Register C holds the length of the datagram to transmit (up to 256 words).
   Register A will be set to 0 if the datagram is successfully queued for transmission, and 1 if there is already a datagram being transmitted.
 - **0x0003**: Configure radio:
   Register B holds the channel to tune to, from 0x0000 through 0x0FF.
   Register C holds the transmit power level to use, from 0x0000 through 0x0007.
   Register A will be set to 0 if the settings are accepted an applied, and 1 if the proposed settings are invalid.
 - **0x0004**: Configure interrupts:
   Register B contains the interrupt message to send when a datagram is received, or 0x0000 to disable receive interrupts.
   Register C contains the interrupt message to send when a datagram transmission completes, or 0x0000 to disable transmit interrupts.


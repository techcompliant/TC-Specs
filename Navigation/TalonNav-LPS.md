```

   \\ TALON NAVIGATION //
   "walk among the stars"
   
```

DCPU-16 Hardware Info:

|     Item       |   Value    |   Comment
| -------------: | ---------- | ----------------
|    Vendor code | 0x982d3e46 | TALON NAVIGATION SYSTEMS
|      Device ID | 0xfce24728 | Local Positioning System (navigator)
|        Version | 0xc59f     |


Description:
    LPS is an entirely local positioning system, using a beacon and receiver
    setup. This device will lock onto the nearest beacon with a specified ID,
    using a doppler reciever to triangulate both position and velocity relative
    to the beacon.

    The Talon LPS has a massive range of 16km, and is accurate to the nearest mm.
    Guaranteed to lock on in under 5 seconds or your money back!
(Note: Guarantee applies only in free space with fully functioning Talon Navigation
Systems LPS Beacon within 16km of receiver, presence of multiple beacons with same ID
may lead to undefined behaivour and voids warranty.)

Interrupt behavior:
    When a HWI is received by the LPS, it reads the A register and does one
    of the following actions:

    0: SET_TARGET_ID
       Sets the target beacon ID to the value of the B register.

    1: GET_LOCAL_COORDINATES
       Stores the current position data to DCPU-16 registers X and Y.

       Depending on the value of the B register, stores a value in X and Y,
       encoded as an INT32 with least significant part in X.

       B value: Value returned
         0: Relative x-coordinate position in mm
         1: Relative y-coordinate position in mm
         2: Relative z-coordinate position in mm

         3: Relative x-coordinate velocity in mm/s
         4: Relative y-coordinate velocity in mm/s
         5: Relative z-coordinate velocity in mm/s


       Register C is set with the operation status:
          0x0000       : Success
          0x0001       : Success, but reduced accuracy
          0x8000       : No fix
          0xffff       : Equipment malfunction


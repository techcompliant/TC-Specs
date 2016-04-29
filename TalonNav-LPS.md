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
    The Talon Navigation Systems LPS is an entirely local positioning system, using 
    a beacon and receiver model. This device uses a doppler receiver to triangulate
    position and velocity relative a specified beacon.

    The Talon has a range of 16km, and is accurate to 1 mm.
    Guaranteed to lock on in under 5 seconds or your money back!

(Note: Guarantee applies only in free space with fully functioning Talon Navigation
Systems LPS Beacon within 16km of receiver. Presence of multiple beacons with same ID
may lead to undefined behaivour and voids warranty.)

Interrupt behavior:
    When a HWI is received by the Talon, it reads the A register and does one
    of the following actions:

    0: Sets the target beacon ID to the value of the B register.

    1: Stores the current position data to memory begining at B.
    
      Data is stored in big endian. B must point to at least 12 words
      of memory. If B+12 is greater or equal to the maximum memory 
      point addressable by the connected CPU, this query will
      silently fail.

      Data is stored in the same layout as this B structure.
      
      struct SpaceCoordinate
      {
         x_high;
         x_low;
         y_high;
         y_low;
         z_high;
         z_low;
         x_velocity_high;
         x_velocity_low;
         y_velocity_high;
         y_velocity_low;
         z_velocity_high;
         z_velocity_low;
      }
      
      All position and velocity information is in the beacon's frame of reference.

      Register C is set with the operation status:
         0x0000       : Success
         0x0001       : Success, but reduced accuracy
         0x8000       : No fix
         0xffff       : Equipment malfunction


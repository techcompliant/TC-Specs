
Device Type Codes
-----

```
 /-------- Hardware class
 |    /--- Hardware sub-class
 |    |                 /--- vendor unique device ID
---- ----           -------------------
0100 1111 1101 0001 0111 1110 1001 0001
          ---- ||||
           |   |||\-- can generate interrupts
           |   ||\--- reset with 0xffff interrupt supported
           |   |\---- memory mapped
           |   \----- supports extended API
           \--- Hardware class API
```

## Hardware Class API

For every hardware class, there may exist a hardware class API.
All devices with the same hardware class and standard API ID can be used in the same generic way.
 * Standard API IDs are 0x0 - 0xD.
 * 0xE means a standard API based on sub-class is used.
 * 0xF means the device does not support standard API


## Hardware Classes and Sub-classes

 * 0 - expansion buses
 * 1 - integrated devices
   * 1 - on-board memory
   * 2 - timers
   * 3 - RNG
   * 4 - real time clocks 
 * 2 - sensors (input only devices)
 * 3 - human interfacing specific devices
   * 0 - keyboards
 * 4 - mass storage
   * 0-7 32+ bit addressable
   * 8-F 16 bit addressable
   * 4 - 32 bit, fixed disk drives
   * A - 16 bit, fixed disk drives
   * F - 16 bit, floppy drives
 * 5 - memory controllers
   * 1 DMA
   * 4 non-volatile memory devices
 * 6 - multimedia capture
 * 7 - raster display (2D)
   * 0 - text cell based, command driven, monochrome display
   * 1 - text cell based, memory mapped, monochrome display
   * 2 - text cell based, command driven, color display
   * 3 - text cell based, memory mapped, color display
   * 4 - pixel based, command driven, monochrome display
   * 5 - pixel based, memory mapped, monochrome display
   * 6 - pixel based, command driven, color display
   * 7 - pixel based, memory mapped, color display
   * 8 - pixel based, memory mapped, color/monochrome video card
 * 8 - vector display
   * 2 - 2D vector
   * 3 - 3D holographic
 * 9 - co-processors/other CPUs/microcontrollers
 * A - reserved/future use
 * B - reserved/future use
 * C - generic output only devices
 * D - wireless devices
 * E - wired network/communication
   * 0 - parallel port
   * 1 - serial port
 * F - non-standard devices


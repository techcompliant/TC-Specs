ROM
----

|     Item       |   Value    |   Comment
| -------------: | ---------- | ----------------
| Device type ID | 0x17400011 | Flashable ROM
|        Version | 1          | dictates size

Device Description
----
The default built in ROM that provides bootloading capacity.

Interrupt Commands
----

 - **0x0000**: INIT
      - 0: Copy the boot sector of the installed disk to memory and execute it.
 - **0x0001**: FLASH
      - B: Flash ROM. B is offset to source


Behaviours
----
Interrupt 0 occurs at boot.


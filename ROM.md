ROM
----

```
--------
```

|     Item       |   Value    |   Comment
| -------------: | ---------- | ----------------
|    Vendor code |            | (Various)
|      Device ID | 0x11F0     | 
|    Device type | 0x11F0     | Integrated memory
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

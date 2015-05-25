ROM
----

```
--------
```

|     Item       |   Value    |   Comment
| -------------: | ---------- | ----------------
|    Vendor code | 0x0000     | TBD
|      Device ID | 0x0000     | TBD
|    Device type | 0x0000     | (TDB)
|        Version | 1          | dicates size

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

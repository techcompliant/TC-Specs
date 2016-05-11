# Silicon Stick Enterprises General-Purpose Integrated Memory Module

```
  _______  _______  _______ 
 |  _____||  _____||  _____|
 | |_____ | |_____ | |_____ 
 |_____  ||_____  ||  _____|
  _____| | _____| || |_____
 |_______||_______||_______|
  Silicon Stick Enterprises
```

|     Item       |   Value      |   Comment
| -------------: | :----------: | :----------------
|    Vendor code | 0x38B8A4D9   | Silicon Stick Enterprises
|      Device ID | _Differs_    | _(see description)_  
|    Device type | 0x17D5       | Integrated memory module.  Generates HWI, memory mapped.
|        Version | 0x0001       | Version 1

# Description
The Silicon Stick Enterprises GPIMM is an integrated module that does little
more than store data for extended periods of time.  The uses for extra memory
are numerous - extending in-built memory, providing small amounts of mobile
storage, or allowing devices to hold information between power cycles.

Due to formatting similarity to Mackapar Media's products, Silicon Stick
Enterprises has licensed use of their access protocols in order to produce a
more streamlined experience for end users.  As such, the modules are
formatted into sectors much like Mackapar Media's, however the modules carry
far fewer to reduce physical size (down to packages as small as 1cm x 1cm x
0.2cm) and improve upon access speed.

These modules are designed with access speed in mind; while all GPIMMs act
asynchronously as Mackapar Media's products do, a full read or write
operation completes within 0.5 milliseconds - an effective read/write speed of
1024 kw/s - and because there are no moving parts, there is no seeking time; 
all data in the module is accessible immediately.

The device comes in two different types in each of four capacities, reflected
in the Device ID.

```
     / ------- Reserved; indicates this is a GPIMM 
     |
  --------------
  0000 0000 0101 0001
                 |--|
                 || |
                 || \ -- Volatile vs Non-Volatile
                 |\ ---- Device Capacity
                 \ ----- Reserved for future capacity options
```

## Volatile vs Non-Volatile
Indicates volitility of the memory stored in the module; in other words, when
power is lost to the module, whether or not the data inside is lost.
* A `0` here indicates the module is _**volatile**_, and is suitable only for
short-term data storage.
* A `1` here indicates the module is _**non-volatile**_, and is suitable for
longer-term data storage.

## Device Capacity
Indicates the storage capacity of the module, per the following table:

| Bits | Size (16-bit words / 512-word Mackapar sectors / KiB)
| :--: | -----------------------
| `00` | 1024 w / 2 sectors / 2 KiB
| `01` | 2048 w / 4 sectors / 4 KiB
| `10` | 4096 w / 8 sectors / 8 KiB
| `11` | 8192 w / 16 sectors / 16 KiB

# Commands

When any GPIMM recieves an `HWI`, the module will read the interrupting device's
`A` Register and performs a function relating to the value found there.  Unless
otherwise stated, all addresses are 16-bit.

### `0x0000`: Poll Device
Sets the interrupting device's `B` Register to the current state (see below)
and the device's `C` Register to the last error since the last device poll. 

### `0x0001`: Set Interrupt
Enables interrupts and will set the message to the value found in the
interrupting device's `X` Register, provided it's any value other than zero.
If the `X` Register is found to be zero, interrupts are disabled on this module.

If interrupts are enabled, the GPIMM will trigger an interrupt on any connected
DCPU-16 devices whenever the state or error message changes on the module.

### `0x0002`: Read Sector
The module examines the interrupting device's `X` and `Y` Registers, and will
read sector `X` into the device's RAM starting at position `Y`.  It will set the
device's `B` Register to 1 if reading has begun, and any other value if reading
has failed.  Reading is only possible if the state is STATE_READY.  The module
will protect against partial reads.

### `0x0003`: Write Sector
The module examines the interrupting device's `X` and `Y` Registers, and will
write sector `X` from the device's RAM starting at position `Y`.  It will set the
device's `B` Register to 1 if writing has begun, and any other value if writing
has failed.  Writing is only possible if the state is STATE_READY.  The module
will protect against partial writes.

# State Codes

|   Value   |     Code                |  Description
| --------: | ----------------------- | -------------
| `0x0000`  | STATE_NO_MEDIA (unused) | The module's memory is always accessible, therefore this state is never reached.
| `0x0001`  | STATE_READY             | The module is ready to accept commands.
| `0x0002`  | STATE_READY_WP (unused) | Because the module cannot be set to read-only mode, this state is never reached.
| `0x0003`  | STATE_BUSY              | The module is currently reading or writing to memory.

# Error Codes

|   Value   |     Code          |  Description
| --------: | ----------------  | ------------------
| `0x0000`  | ERROR_NONE        | There has been no error since the last poll.
| `0x0001`  | ERROR_BUSY        | The module is busy performing an action.
| `0x0002`  | ERROR_BAD_ADDRESS | Attempted to read from a sector this module doesn't contain.
| `0x0003`  |                   | Reserved. 
| `0x0004`  |                   | Reserved.
| `0x0005`  | ERROR_BAD_SECTOR  | The physical space relating to the requested sector is destroyed, and the data on it is lost.  Replacement of the module is recommended.
| `0xFFFF`	| ERROR_BROKEN	    | Theres been some major software or hardware problem.  Perform a power cycle to recover the module; if that doesn't work, replacement is required.
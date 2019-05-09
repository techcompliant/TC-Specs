# KCM-11 Memory Management Unit

|     Item       |   Value     |   Comment
| -------------: | ----------- | ----------------
|    Vendor code | 0x9CC16DAB  | Kingston Chronometers and Memorabilia
|      Device ID | 0x58E2110B  | KCM-11 MMU
|    Device type | 0x58E2      | Memory management unit (MMU)
|        Version | 0x000B (11) |


The KCM-11 is a hardware memory management unit that enables the 16-bit address
space of the DCPU-16 (or its RISC competitor the
[Risque-16](https://github.com/shepheb/risque16)) to address more than 64KiW of
physical memory.

The KCM-11 has 16-bit segment numbers, allowing 65,536 (64K) segments. Since
each segment is 4KiW, this allows a maximum physical memory of 256MiW. Of course
most DCPU-16 configurations have much less than that!

## User Guide

The KCM-11 divides physical memory into 4KiW segments, each with a segment
number. These segments of physical memory are mapped into the virtual address
space based on a translation table.

This table resides inside the KCM-11 hardware, and is adjusted using `HWI`s.

The KCM-11's work is invisible to the running processor - the CPU addresses
64KiW like it always does, and these virtual addresses are transparently mapped
to physical addresses by the KCM-11.

### Translation Table

With 4KiW segments and 64KiW of address space, there are only 16 segments to map.
Therefore the KCM-11 contains 16 hardware registers, each holding the physical
segment number to map to that virtual address range.

This mapping is updated with `HWI`s, see below.

### Mapping a Nonexistent Segment

If a `HWI` requests mapping to a physical segment that does not exist, the `A`
register is set to 0 to indicate this failure. However, this is a "soft"
failure: the KCM-11 will still be working normally otherwise.

Reading from a nonexistent segment always returns `0xFFFF`; writes to
nonexistent segments silently have no effect.

### Startup State

At startup, the 16 segments of virtual memory are mapped to the first 16
segments of physical memory.

Therefore a computer equipped with a KCM-11 can be treated at startup as though
no MMU exists, and you get a normal 64KiW address space.

### Interaction with Memory Mapped Hardware

Memory-mapped hardware, like the LEM1802 VRAM, interacts with the KCM-11 in a
straightforward way.

The address given to the other hardware is a *virtual* address, within the
normal 64KiW address space. The values at those virtual addresses are, however,
subject to translation.

So if you change the KCM-11 translation table, other hardware devices will
continue mapping to the same virtual address, but the values will change to
match those found on the corresponding segment of physical memory.

Hardware devices that write to a memory-mapped region will end up writing their
data to two different segments, if the segment is switched mid-stream.

### Mirroring

It is legal and supported to map a single physical segment to multiple virtual
segments. Then the same data is found at multiple virtual addresses.

## Interrupt Commands

Select the desired interrupt based on the `A` register.

- **0x0000**: `MAP_SEGMENT`
    - Maps virtual segment number `B` to physical segment number `C`.
    - Sets `A` to 1 on success, 0 if the requested segment does not exist.
- **0x0004**: `MEMORY_INFO`
    - Sets `B` to the number of 4KiW physical segments available.
    - A slight quirk: A value of 0 indicates `0x10000`, the maximum.
- **0xFFFF**: `RESET`
    - Resets the memory map to the startup state: mapping directly to the first
      64KiW of physical memory.


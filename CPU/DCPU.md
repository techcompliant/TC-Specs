# DCPU-TC Spec - Preliminary draft

## Overview

Memory consists of 2<sup>16</sup> (65536) 16-bit words.

Registers are also 16-bit:
- 8 general purpose registers: `A`, `B`, `C`, `X`, `Y`, `Z`, `I` and `J`.
- Program counter: `PC`
- Stack pointer: `SP`
- Excess: `EX`
- Interrupt address: `IA`

## Operands

An operand is either a literal value, or a source/destination location where a value is stored.
Locations are either a register, or a memory location, denoted `[address]`.

Operands are represented as follows:

| Cycles | Value       | Description
| -------|-------------|------------
| 0      | 0x00 - 0x07 | `A`, `B`, `C`, `X`, `Y`, `Z`, `I`, `J`
| 0      | 0x08 - 0x0F | `[register as above]`
| 1      | 0x10 - 0x17 | `[register + additional word]`
| 0      | 0x18        | `[--SP]` or `PUSH` when writing, `[SP++]` or `POP` when reading
| 0      | 0x19        | `[SP]` or `PEEK`
| 1      | 0x1A        | `[SP + additional word]`, also written `PICK n`
| 0      | 0x1B        | `SP`
| 0      | 0x1C        | `PC`
| 0      | 0x1D        | `EX`
| 1      | 0x1E        | `[additional word]`
| 1      | 0x1F        | additional word (literal)
| 0      | 0x20 - 0x3F | literal value 0xFFFF - 0x1E (-1 to 30) (literal)

### Notes

- Trying to write to a literal silently does nothing.
- `PICK 0` is equivalent to `PEEK`, but takes an extra word and cycle.
- Inline literals (0x20 - 0x3F) are only possible in the 6-bit `a` operand; see below.
- Instructions that ignore their operand (eg. `RFI`) treat it as a read, and discard the value.
- See the [Errata](https://github.com/techcompliant/TC-Specs/wiki/DCPU-Errata#operand-combinations) for details on obscure combinations, like what `SET SP, POP` does.


## Instructions

An instruction consists of one to three words. The first word contains the opcode and two operands, in the format above. This first word is followed by the extra word required by the `a` operand, if any, then the extra word for the `b` operand, if any.

If the lower 5 bits are all 0, then it is a single-operand instruction, otherwise it is a dual-operand instruction.

The number of cycles taken to execute an instruction is equal to the cycles”of the instruction as given in the below tables, plus the sum of the cycles”of the operands as given in the above table.
Even when the `b` operand is being read and written, its cycle cost is only paid once.

### Dual-operand instructions:

Dual-operand instructions have the following format (MSB->LSB):

“a” operand | “b” operand” | opcode
------------|--------------|-------
6 bits      | 5 bits       | 5 bits

Note that only the 6-bit `a` operand can hold inline literals (0x20-0x3f).

| Cycles | Opcode | Mnemonic   | Description
| -------|--------|------------|------------
|        | 0x00   |            | (Single-operand instruction)
| 1      | 0x01   | `SET b, a` | `b = a`
| 2      | 0x02   | `ADD b, a` | `b = b + a`, `EX = 1` if overflow, `0` otherwise
| 2      | 0x03   | `SUB b, a` | `b = b - a`, `EX = 0xFFFF` if underflow, `0` otherwise
| 2      | 0x04   | `MUL b, a` | `b = b * a`, `EX = ((b*a)>>16)&0xFFFF`, `a` and `b` are unsigned
| 2      | 0x05   | `MLI b, a` | `b = b * a`, `EX = ((b*a)>>16)&0xFFFF`, `a` and `b` are signed
| 3      | 0x06   | `DIV b, a` | `b = b / a`, `EX = ((b<<16)/a)0xFFFF`, if `a == 0` sets `b` and `EX` to 0; `a` and `b` are unsigned
| 3      | 0x07   | `DVI b, a` | As `DIV`, with `a` and `b` signed. Rounds towards 0 (see below).
| 3      | 0x08   | `MOD b, a` | `b = b % a`, if `a == 0` sets `b` to 0, never sets `EX`; `a` and `b` are unsigned
| 3      | 0x09   | `MDI b, a` | As `MOD`, with `a` and `b` signed. Rounds towards 0 (see below).
| 1      | 0x0A   | `AND b, a` | `b = b & a`
| 1      | 0x0B   | `BOR b, a` | `b = b | a`
| 1      | 0x0C   | `XOR b, a` | `b = b ^ a`
| 1      | 0x0D   | `SHR b, a` | `b = b >>> a`, `EX = ((b << 16) >> a) & 0xFFFF`, logical shift (`b` unsigned)
| 1      | 0x0E   | `ASR b, a` | `b = b >> a`, `EX = ((b << 16) >>> a) & 0xFFFF`, arithmetic shift (`b` signed)
| 1      | 0x0F   | `SHL b, a` | `b = b << a`, `EX = ((b << a) >> 16) & 0xFFFF`
| 2\*    | 0x10   | `IFB b, a` | skip next instruction unless `b & a != 0`
| 2\*    | 0x11   | `IFC b, a` | skip next instruction unless `b & a == 0`
| 2\*    | 0x12   | `IFE b, a` | skip next instruction unless `b == a`
| 2\*    | 0x13   | `IFN b, a` | skip next instruction unless `b != a`
| 2\*    | 0x14   | `IFG b, a` | skip next instruction unless `b > a`, `a` and `b` unsigned
| 2\*    | 0x15   | `IFA b, a` | skip next instruction unless `b > a`, `a` and `b` signed
| 2\*    | 0x16   | `IFL b, a` | skip next instruction unless `b < a`, `a` and `b` unsigned
| 2\*    | 0x17   | `IFU b, a` | skip next instruction unless `b < a`, `a` and `b` signed
|        | 0x18   |            |
|        | 0x19   |            |
| 3      | 0x1A   | `ADX b, a` | `b = b + a + EX`, `EX = 1` if overflow, 0 otherwise
| 3      | 0x1B   | `SBX b, a` | `b = b - a + EX`, `EX = 0xffff` if underflow, 0 otherwise
|        | 0x1C   |            |
|        | 0x1D   |            |
| 2      | 0x1E   | `STI b, a` | `b = a`, `I++`, `J++` (does not change `EX`)
| 2      | 0x1F   | `STD b, a` | `b = a`, `I--`, `J--` (does not change `EX`)

#### Branching instructions

\* Branching instructions take 2 cycles when the check passes, and 3 when skipping.

When skipping, if the next instruction is also a branching instruction, skipping
continues at the cost of 1 extra cycle. This allows branching instructions to be
"chained", with the next non-branching instruction executed only if all the
conditions are true.

#### On Signed Division

Short answer: Quotients are always rounded toward 0.

For example, `-7 / 2` is `-3.5`, which rounds to `-3`. `-3 * 2 = -6`, so the
remainder is `-1`.

For a more detailed discussion with extra examples, see the [Errata](https://github.com/techcompliant/TC-Specs/wiki/DCPU-Errata#signed-division).

#### EX

Note that `EX` is written last, for operations that set it. This is important
when `EX` is also an argument.

The general order is: read `a`, read `b`, write `b`, write `EX`.

See the [Errata](https://github.com/techcompliant/TC-Specs/wiki/DCPU-Errata#operand-combinations) for examples.


### Single-operand instructions:

Single-operand instructions have the following format (MSB->LSB):

“a” operand | opcode | 00000
------------|--------|-------
6 bits      | 5 bits | 5 bits

| Cycles | Opcode | Mnemonic | Description
| -------|--------|----------|-------------
|        | 0x00   |            | Reserved
| 3      | 0x01   | `JSR a`    | Pushes `PC`, then sets `PC = a`
|        | 0x02   |            | 
|        | 0x03   |            | 
|        | 0x04   |            | 
|        | 0x05   |            | 
|        | 0x06   |            | 
|        | 0x07   |            | 
| 4      | 0x08   | `INT a`    | Software interrupt with message `a`
| 1      | 0x09   | `IAG a`    | Sets `a = IA`
| 1      | 0x0A   | `IAS a`    | Sets `IA = a`
| 3      | 0x0B   | `RFI a`    | Return from interrupt: Pop `A`, pop `PC`, disable interrupt queuing (note `a` is not actually used)
| 2      | 0x0C   | `IAQ a`    | `a != 0`: interrupts are queued; `a == 0`: interrupts trigger normally
|        | 0x0D   |            | 
|        | 0x0E   |            | 
|        | 0x0F   |            | 
| 2      | 0x10   | `HWN a`    | Sets `a` to the number of connected hardware devices
| 4      | 0x11   | `HWQ a`    | Queries hardware device `a` for information. `A+(B<<16)` = hardware ID, `C` = hardware version, `X+(Y<<16)` = manufacturer ID
| 4+     | 0x12   | `HWI a`    | Send interrupt to hardware device `a`
| 1      | 0x13   | `LOG a`    | Send value `a` to the log system
|        | 0x14   | `BRK a`    | Send value `a` to the’conditional break system
|        | 0x15   | `HLT a`    | Halts execution of code until resumed by hardware interrupt (note a is ignored; see section on Interrupts)
|        | 0x16   |            | 
|        | 0x17   |            | 
|        | 0x18   |            | 
|        | 0x19   |            | 
|        | 0x1A   |            | 
|        | 0x1B   |            | 
|        | 0x1C   |            | 
|        | 0x1D   |            | 
|        | 0x1E   |            | 
|        | 0x1F   |            | 

## Interrupts:

Interrupts are generated by hardware, or by `SWI`. An interrupt has a 16-bit *message*.
When an interrupt is generated, it goes into the queue (even if queueing is off).
This allows for multiple devices to generate interrupts at the same time,
without dropping any.

When the DCPU goes to execute an instruction, it first checks for interrupts. If
interrupt queueing is off, and there is at least one interrupt in the queue,
that interrupt is triggered.

When an interrupt is triggered, the DCPU checks `IA`. If `IA = 0`, the interrupt
is discarded and execution continues normally. If `IA != 0`, the DCPU processes
the interrupt as follows:

1. Turn on interrupt queueing
2. Push `PC` (the instruction we were about to execute)
3. Push register `A`
4. Set `PC = IA`
5. Set `A` to the interrupt message

Execution then continues at `IA`.

### Notes

- The `RFI` instruction is designed to neatly reverse the process above:
  it pops `A` and `PC`, and turns interrupt queuing off.
- Conversely, `RFI` is not magic: you can return manually from an interrupt
  handler with other instructions.
- Only one interrupt is popped at a time. If `IA = 0`, one interrupt is
  discarded for each instruction executed.
- The maximum length of the interrupt queue is 256. When the 257th interrupt is
  added, the DCPU's behavior is undefined. (Generally, memory is corrupted
  unpredictably.)
    - Note that older models often caught fire in this condition; that hardware
      fault has been corrected, but the behavior is still undefined.
- Interrupts are not checked while "skipping" due to a branch instruction, even
  if several instructions are skipped. Interrupts only trigger when instructions
  are actually executed.
- Entering an interrupt does not cost any extra cycles.

### Nesting Interrupts

Possible, with care. You are free to turn interrupt queueing off from inside an
interrupt handler.

Similarly, you are free to return from an interrupt without turning queueing
off.

## Hardware

A DCPU-TC can have up to 0xFFFF (65535) attached hardware devices. Devices are
numbered from 0. Communication with these devices is achieved with interrupts.

Upon receiving an interrupt with `HWI`, a hardware device can read/write to the
DCPU-TC’s memory and registers. A device may not send an interrupt to the DCPU-TC
or modify its memory and registers before it has received at least one interrupt
from the DCPU-TC.

## Debugging

DCPU-TC provides two main provisions for debugging - `LOG` and `BRK`.

`LOG` will output a value to the computer's logging system, which is
generally some output buffer displayed using an external device.

`BRK` outputs a value to the computer's break system. This system
performs some test on the value (eg, equality to some other value)
and conditionally halts the DCPU-TC’s execution to allow for
inspection of memory etc.

Exactly how this debugging instructions work is dependent on the exact
configuration of the DCPU system. In the simplest case, they may do nothing at
all.

## Halt and Interrupt Queueing

What happens when `HLT` is executed while interrupt queueing is on?

Short answer: the DCPU is blocked forever, and must be hardware reset.

In more detail, `HLT` blocks the DCPU until an interupt is *triggered*, not
*generated*. Hardware devices and `SWI` *generate* interrupts that go in the
queue, but it's only when interrupts *trigger* on the way out of the queue that
the DCPU breaks out of `HLT` state.

Since interrupts are never *triggered* when queueing is on, the DCPU is halted
forever.

Note that `HLT` state ends when an interrupt is triggered, even if `IA = 0` and
the triggered interrupt gets discarded.


## Initial State

On startup, the DCPU is in the following state:

- All registers are 0 - including `PC`, `IA` and `SP`.
    - This implies that the first instruction to be executed is at 0.
    - Note that `SP` is decremented before storing, so the first value pushed
      goes in `[0xffff]`.
- ROM is copied to the begining of memory.
- All other memory is undefined - it may be 0, but it also may not.
- Interrupt queueing is off (interrupts will flow, but `IA` is 0 so they'll be
  discarded).
- All hardware devices are in their initial state (no memory mapped, etc.)

That's true on a cold start, or a reset.


DCPU-TC Spec - Preliminary draft
======

Overview:
------


Memory consists of 2<sup>16</sup> (65536) 16bit words.

Registers are also 16-bit:
- 8 general purpose registers: A, B, C, X, Y, Z, I and J.
- Program counter: PC
- Stack pointer: SP
- Excess: EX
- Interrupt address: IA

Operands:

An operand represents either a value, or a source/destination location where a value is stored(a register, or memory location denoted [address] )).

Values are mapped as follow:

Cycles | Value       | Description
-------|-------------|------------
0      | 0x00 - 0x07 | A, B, C, X, Y, Z, I, J
0      | 0x08 - 0x0F | [register as above]
1      | 0x10 - 0x17 | [register + additional word]
0      | 0x18        | [--SP] if destination (being written to), [SP++] if source (being read from)
0      | 0x19        | [SP]
1      | 0x1A        | [SP + additional word]
0      | 0x1B        | SP
0      | 0x1C        | PC
0      | 0x1D        | EX
1      | 0x1E        | [additional word]
1      | 0x1F        | additional word (literal)
0      | 0x20 - 0x3F | literal value 0xFFFF - 0x1E (-1 to 30) (literal)

Instructions:

An instruction may consist of one to three words. The first word contains the opcode and any operands. The second word is the “additional word” of an “a” operand and the third word is the “additional word” of a “b” operand.

If the lower 5 bits are all ‘0’, then it is a single-operand instruction, otherwise it is a dual-operand instruction.

The number of cycles taken to execute an instruction is equal to the “Cycles” of the instruction as given in the below tables, plus the sum of the “Cycles” of the operands as given in the above table.

Dual-operand instructions:
==========
Dual-operand instructions have the following format (MSB->LSB):

“a” operand | “b” operand” | opcode
------------|--------------|-------
6 bits      | 5 bits       | 5 bits

Table of dual-operand instructions:
----------

|Cycles | Opcode | Mnemonic | Description
|-------|--------|----------|------------
|       | 0x00   |          | Single-operand instruction
|1      | 0x01   | SET b, a | b = a
|2      | 0x02   | ADD b, a | b = b + a, if overflow: EX = 0x0001
|2      | 0x03   | SUB b, a | b = b - a, if underflow: EX = 0xFFFF
|2      | 0x04   | MUL b, a | b = b * a, EX = ((b*a)>>16)&0xFFFF, a and b are unsigned
|2      | 0x05   | MLI b, a | b = b * a, EX = ((b*a)>>16)&0xFFFF, a and b are signed
|3      | 0x06   | DIV b, a | b = b / a, EX = ((b<<16)/a)0xFFFF, if a == 0: b = 0, EX = 0, a and b are unsigned
|3      | 0x07   | DVI b, a | b = b / a, EX = ((b<<16)/a)0xFFFF, if a == 0: b = 0, EX = 0, a and b are signed
|3      | 0x08   | MOD b, a | b = b % a, if a == 0: b = 0, a and b are unsigned
|3      | 0x09   | MDI b, a | b = b % a, if a == 0: b = 0, a and b are signed
|1      | 0x0A   | AND b, a | b = b & a
|1      | 0x0B   | BOR b, a | b = b \| a
|1      | 0x0C   | XOR b, a | b = b ^ a
|1      | 0x0D   | SHR b, a | b = b >>> a, EX = ((b<<16)>>a)&0xFFFF, logical shift
|1      | 0x0E   | ASR b, a | b = b >> a, EX = ((b<<16)>>>a)&0xFFFF, arithmetic shift, b is signed
|1      | 0x0F   | SHL b, a | b = b << a, EX = ((b<<a)>>16)&0xFFFF
|2*     | 0x10   | IFB b, a | skip next instruction unless b & a != 0
|2*     | 0x11   | IFC b, a | skip next instruction unless b & a == 0
|2*     | 0x12   | IFE b, a | skip next instruction unless b == a
|2*     | 0x13   | IFN b, a | skip next instruction unless b != a
|2*     | 0x14   | IFG b, a | skip next instruction unless b > a
|2*     | 0x15   | IFA b, a | skip next instruction unless b > a, a and b are signed
|2*     | 0x16   | IFL b, a | skip next instruction unless b < a
|2*     | 0x17   | IFU b, a | skip next instruction unless b < a, a and b are signed
|       | 0x18   |          | 
|       | 0x19   |          | 
|3      | 0x1A   | ADX b, a | b = b + a + EX, if overflow: EX = 0x0001
|3      | 0x1B   | SBX b, a | b = b - a + EX, if underflow: EX = 0xFFFF
|       | 0x1C   |          | 
|       | 0x1D   |          | 
|2      | 0x1E   | STI b, a | b = a, I++, J++
|2      | 0x1F   | STD b, a | b = a, I--, J--

*Branching instructions take 3 cycles on a skip, and 2 cycles
otherwise. If a branching instruction skips a branching instruction,
it will continue skipping, adding a cycle’s delay each time. This
effectively combines consecutive conditionals into their logical
conjunction.

Single-operand instructions:
========
Single-operand instructions have the following format (MSB->LSB):

“a” operand | opcode | 00000
------------|--------|-------
6 bits      | 5 bits | 5 bits
Table of single-operand instructions:

|Cycles | Opcode | Mnemonic | Description
|-------|--------|----------|-------------
|       | 0x00   |          | Reserved
|3      | 0x01   | JSR a    | [SP] = PC + 1, PC = a
|       | 0x02   |          | 
|       | 0x03   |          | 
|       | 0x04   |          | 
|       | 0x05   |          | 
|       | 0x06   |          | 
|       | 0x07   |          | 
|4      | 0x08   | INT a    | software interrupt with message a
|1      | 0x09   | IAG a    | a = IA
|1      | 0x0A   | IAS a    | IA = a
|3      | 0x0B   | RFI a    | disable interrupt queuing, A = [SP++], PC = [SP++]; (note a is not actually used)
|2      | 0x0C   | IAQ a    | if a != 0: interrupts go to queue, if a == 0: intterupts trigger normally
|       | 0x0D   |          | 
|       | 0x0E   |          | 
|       | 0x0F   |          | 
|2      | 0x10   | HWN a    | a = number of connected hardware devices
|4      | 0x11   | HWQ a    | (A, B, C, X, Y) = info about hardware device a, A+(B<<16) = hardware id, C = hardware version, X+(Y<<16) = manufacturer id
|4+     | 0x12   | HWI a    | send interrupt to hardware device a
|1      | 0x13   | LOG a    | send value a to computer’s log system
|       | 0x14   | BRK a    | send value a to computer’s conditional break system
|       | 0x15   | HLT a    | Halts execution of code until resumed by external interrupt (note a is ignored)
|       | 0x16   |          | 
|       | 0x17   |          | 
|       | 0x18   |          | 
|       | 0x19   |          | 
|       | 0x1A   |          | 
|       | 0x1B   |          | 
|       | 0x1C   |          | 
|       | 0x1D   |          | 
|       | 0x1E   |          | 
|       | 0x1F   |          | 
       
Interrupts:
===========
Only one interrupt can be triggered for each instruction executed.
When an interrupt is generated, it is triggered immediately if the
queue is empty and no other interrupts have occurred this
instruction, otherwise it is added to the back of the queue.
The front of the queue is triggered if no other interrupts have been
triggered this instruction.

If IA is set to 0, interrupts will do nothing when triggered, otherwise:
- Turn on interrupt queuing
- [--SP] = PC
- [--SP] = A
- PC = IA
- A = interrupt message

Hardware:
=========
A DCPU-TC can have up to 0xFFFF (65535) attached hardware devices.
Communication to these devices is via interrupts. Upon receiving an
interrupt, a hardware device can read/write to the DCPU-TC’s memory 
and registers. A device may not send an interrupt to the DCPU-TC
before it has received at least one interrupt from the DCPU-TC.

Debugging:
==========
DCPU-TC provides two main provisions for debugging - LOG and BRK.

LOG will output a value to the computer’s logging system, which is
generally some output buffer displayed using an external device.

BRK outputs a value to the computer’s break system. This system
performs some test on the value (eg, equality to some other value)
and conditionally halts the DCPU-TC’s execution to allow for
inspection of memory etc.

# Generic Random Number Generator

|     Item       |   Value    |   Comment
| -------------: | ---------- | ----------------
|    Vendor code | 0          | Generic
|      Device ID | 0x13e2dc12 | Generic RNG
|    Device Type | 0x13e2     | Standard RNG
|        Version | 1          |

## Device Description

Generic (pseudo)random number generator. Has a built-in source of entropy, and
can be seeded either from that source or from a software-supplied seed.

At power-on, or after a `RESET` (`0xffff`), the device is seeded by the internal
entropy source and ready to generate random numbers. Software **need not**
provide a seed.

## Interrupt Commands

 - **0x0000**: `SET_SEED`
 	- B: Seed value is set to B. If B is 0, seed from the internal entropy
          source.
 - **0x0001**: `GET_RANDOM`
 	- C: Set to a random number, uniformly from 0 to 0xffff.
 - **0xffff**: `RESET`
	- Resets to initial state: seeded from the internal entropy source and
          ready to generate pseudorandom numbers.


Generic Keyboard 
----

```
---=I NEED A LOGO=---
```

|     Item       |   Value    |   Comment
| -------------: | ---------- | ----------------
|    Vendor code | 0x1c6c8b36 | NYA Elektriska
|      Device ID | 0x30cf7406 | Generic Keyboard
|    Device type | 0x0000     | (TBD)
|        Version | 1          | Fixed Version

A generic input keyboard.

Interrupt Commands
----

- **0x0000**: CLEAR_BUFFER
	- 0: Clears the keyboard buffer
- **0x0001**: GET_NEXT
	- C: Store next key from buffer to C register, or 0 if the buffer is empty
- **0x0002**: CHECK_KEY
	- B+C: Check if key in B is current pressed. Set C to result (1 = true, 0 =false)
- **0x0003**: SET_INT
	- B: Interrupt message set to B. Zero to disable


 Behaviours
 ----
When interrupts are enabled, the keyboard will trigger an interrupt when one or
more keys have been pressed, released, or typed.

Key numbers are:
	0x10: Backspace
	0x11: Return
	0x12: Insert
	0x13: Delete
	0x20-0x7f: ASCII characters
	0x80: Arrow up
	0x81: Arrow down
	0x82: Arrow left
	0x83: Arrow right
	0x90: Shift
	0x91: Control

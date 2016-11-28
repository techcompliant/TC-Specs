Generic Keyboard 
----

```
---=I NEED A LOGO=---
```

|     Item       |   Value    |   Comment
| -------------: | ---------- | ----------------
|    Vendor code | 0x1c6c8b36 | NYA Elektriska
|  Compatible ID | 0x30cf7406 | Generic Keyboard (older models)
| Device type ID | 0x30c17406 | Generic ASCII Keyboard
|        Version | 1          | Fixed Version

A generic input keyboard.  All input is buffered in an 8 item queue, excess keys presses push the earliest keypress off the queue.  In addition, the "CHECK_KEY" functionality will only work with up to 8 keys pressed simultaneously.

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
- **0x0004**: SET_MODE
	- B: Keyboard mode set to B(0 = smart text, 1=generic keycode).  Buffer is cleared.

Behaviours
----
The keyboard driver is capable of operating in 2 distinct modes: Smart text input, and generic keycode input.  The keyboard driver defaults to Smart text input, but can be switched to either mode via interrupt 0x0004.

Smart Text Mode
===============
While in smart text mode, the keyboard driver will automatically parse incoming keypresses, and yield a stream of character events.  These events will be fully translated, so "shift-a" will be sent as "A".  Unfortunately, the keyboard driver is unable to give keydown/up status for more then a few specific keys in this state, notably the "modifier" keys: ctrl, shift, alt.  The interrupt will fire only when a new event is added to the key buffer.

In addition to standard alphanumerics and symbols, the keyboard driver will also encode specific additional keys as follows:
-	0x10: Backspace
-	0x11: Return
-	0x12: Insert
-	0x13: Delete
-	0x20-0x7f: ASCII characters
-	0x80: Arrow up
-	0x81: Arrow down
-	0x82: Arrow left
-	0x83: Arrow right
-	0x90: Shift
-	0x91: Control
-	0x92: Alt

- ASCII letters are uppercase when shift is held down.

Generic Keycode Mode
====================
While in generic keycode mode, the keyboard driver will not parse incoming key events, and instead will queue them unmodified to the key buffer.  It will encode both key down and key up events, with the highest bit, 0x8000, set if the event is a key up, with the interrupt firing on every addition to the buffer.

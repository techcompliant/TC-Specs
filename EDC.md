Embedded Display Controller
----

```
         ____   _____________    _____
        /    | /     /  ____/ _____  /
       /     |/     /  /________  /
      /   /|     /|/  __/_   __/
     /___/ |____/ /__/|__/__/   ___
    /___/  |___/ /______/__/ ___  /
   /___/__________|___|/__/__  /
  /_________________/_______/

```

|     Item       |   Value    |   Comment
| -------------: | ---------- | ----------------
|    Vendor code | 0x59EA5742 | Meisaka Engineering and Integration
|      Device ID | 0x70E3E4FF | EDC
|    Device type | 0x70E3     | Text cell based, command driven, monochrome display
|        Version | various    | Determines display characteristics, see below.

The EDC or *Embedded Display Controller* is a small, command based controller and driver for small cell based text screens.
Several models of EDC are available to drive both low power liquid crystal displays (LCD), as well as brighter vacuum fluorescent display (VFD) screens.

The EDC comes in two different feature sets:
 - Basic module feature set EDC: only acts as a display driver.
 - Advanced modules feature set EDC: includes both the display controller/driver and input controller.

Basic/Advanced EDC display driver/controller features:
 - 128 programmable glyphs.
 - 8 x 5 pixel text characters with built-in ASCII compatible character set.
 - Automatic blinking prompt control.
 - up to 4 lines and 124 column display modules supported.
 - built-in display RAM (write only by host device)
 - simple command driven interface with both serial and DCPU options available.

Advanced EDC input controller features:
 - full 4 x 4 matrix keypad decoder with automatic scanning.
 - 3 line accessory function/power control bits.
 - 16 word multiplex registers for extended/more advanced accessories.
 - utilizes the same simple command driven interface as the display.
 - programmable transmit/interrupt for keypad.

Interrupt Commands
----

The display controller may be interfaced directly to a DCPU or connected via a serial interface.
When connected to a DCPU, register A will hold the command index, and register B will hold the first parameter value, C and X hold parameters 2 and 3 respectively for advanced mode commands.
When connected to a serial interface, the first word (or high word of 32 bit message) sent to the EDC acts as the command index, and the second word (or 32 bit low word) sent is the parameter value (if required).

- **0x0000**: DISPLAY_CONTROL
    - Display options depend on each individual bit of the parameter.
    ```
    MSB ---- ---- LSB
    ....0000 ...00000
        ||||    ||||\
        ||||    |||\ Display power: 0 is off, 1 is on
        ||||    ||\ Backlight power: 0 is off, 1 is on (ignored if no backlight present)
        ||||    |\ Cursor display: 0 is off, 1 is on
        ||||    \ Cursor blink: 0 is steady, 1 is blink
        |||\     Cursor style: 0 is block, 1 is underscore
        ||\ Keypad enable (Advanced module)
        |\ Accessory 1 enable (Advanced module)
        \ Accessory 2 enable (Advanced module)
         Accessory 3 enable (Advanced module)

    Other bits are ignored by the display.
    ```
- **0x0001**: CURSOR_ADDRESS
    - Sets the cursor address to the value of the parameter.
        - Line 0 always starts at 0x0000
        - Line 1 always starts at 0x0080
        - Line 2 always starts at 0x0100
        - Line 3 always starts at 0x0180
        - Glyph RAM starts at 0x0200 and ends at 0x0400
        - Values above 0x03FF are ignored and do not affect the display
        - Values beyond line length will put the cursor offscreen, the next character written will not be visible, and will reset the cursor to the beginning of the next line
        - When the cursor is set to glyph RAM, it will count up each write until it reaches the end of glyph RAM at 0x0400, at that point it will return to 0x0000.
- **0x0002**: RESET_CONTROL
    - The parameter controls what action to take:
        - Bit 0: Clear screen if set
        - Bit 1: Home cursor to address 0 if set.
- **0x0003**: WRITE_DISPLAY
    - The parameter controls what to display:
        - Bits 0-7: Character to display:
            - 0x00-0x7F: built-in font glyphs
            - 0x80-0xFF: character RAM glyphs.
        - Bit 8: Do not advance cursor (overwrite), cursor advances if not set.
        - When cursor is in glyph RAM, writes the entire 16 bit value and advances.
- **0x0080**: READ_KEYPAD (Advanced modules only)
    - reads the state of all keys on the keypad, a 16 bit value represents the keypad matrix, 1 = pressed, 0 = not pressed.
    - All bits will read as 0 when the keypad is disabled, even if the keys are pressed.
    - for consistency, the columns should be wired left to right ascending, rows top to bottom ascending:
        - bit 0 would be the top left most key.
        - bit 3 would be the top right most key.
        - bit 15 would be the bottom right most key.
    - in DCPU mode, this is stored in A.
    - in serial mode, the 16 bit value is simply transmitted.
- **0x0081**: SET_KEYPAD_NOTIFY (Advanced modules only)
    - this function takes a word parameter and saves it as the notification word.
        - if the parameter is zero: disable keypad notification
        - if the parameter is non-zero: enable keypad notification
    - when notification is enabled and the keypad state changes (key press/release) the EDC will notify:
        - for a DCPU interface, the EDC will send an interrupt with the saved word.
        - for serial interface, the EDC will transmit the saved word followed by the new 16 bit keypad state.
- **0x0082**: READ_ACCESSORY (Advanced modules only)
    - takes a word parameter, and uses it as an index to a pair of the accessory registers.
        - index 0 would be the first two registers, 1 would be the next two, and so on.
        - only the lowest 3 bits are used as an index, (0-7 access all 16 registers in pairs), the upper bits are ignored, but should be set to 0 for compatibility.
        - accessory registers are bi-directional, but are not memory: they all read as all ones if no accessory is attached.
    - For DCPU connections: first accessory register value is stored in A, second in B.
    - For serial connections: transmits the first accessory register, hollowed by the second.
- **0x0083**: WRITE_ACCESSORY (Advanced modules only)
    - takes 3 word parameters, the first parameter is an index to a pair of the accessory registers, parameter 2 and 3 are the first and second values (respectively) to write to the accessory registers.
        - index 0 would be the first two registers, 1 would be the next two, and so on.
        - only the lowest 3 bits are used as an index, (0-7 access all 16 registers in pairs), the upper bits are ignored, but should be set to 0 for compatibility.
- **0xFFFE**: QUERY_DISPLAY
    - For serial connections: Causes the display to transmit 5 words containing vendor code, device ID and version
    - For direct DCPU connections: Has the same effect as an HWQ instruction.
- **0xFFFF**: RESTART_DISPLAY
    - Resets all display settings and zeroes RAM.


Behaviours
----

### Version code
Version code determines the display size and characteristics.
```
       MSB ---- ---- LSB
       XXXXMMVI FRRCCCCC
```
* X: Reserved bits (always 0)
* MM: Cell size, always 01: 8x5 pixel cells, with 1 pixel spacing
* VI: Media type:
    * 11: Inverted LCD with backlight (active pixels are backlight color)
    * 10: VFD (active pixels glow VFD color)
    * 01: LCD with backlight (active pixels are dark)
    * 00: Reflective LCD (active pixels are dark).
* F: Firmware version:
    * 0: Basic display module.
    * 1: Advanced display and input module.
* RR: Text cell line count, lines = RR + 1
* CCCCC: Text cell column count, columns = CCCCC * 4.

Common values:
* 0x0402: 1x8 cell, reflective LCD
* 0x052A: 2x40 cell, backlit LCD
* 0x0534: 2x80 cell, backlit LCD
* 0x0625: 2x20 cell, VFD.

### Glyph RAM
* The glyph RAM holds up to 128 custom raster glyphs; each glyph is 4 words
* Glyphs are rasterized by the display horizontally, 8 bits per cell row
* On  the standard 5x8 cell this is:
    * Word 0: Bits 0-4 are row 1, columns 1-5
    * Word 0: Bits 8-12 are row 2, columns 1-5
    * Word 1: Bits 0-4 are row 3, columns 1-5
    * Word 1: Bits 8-12 are row 4, columns 1-5
    * Word 2: Bits 0-4 are row 5, columns 1-5
    * Word 2: Bits 8-12 are row 6, columns 1-5
    * Word 3: Bits 0-4 are row 7, columns 1-5
    * Word 3: Bits 8-12 are row 8, columns 1-5.
* When a WRITE_DISPLAY query is called while the cursor is in the glyph RAM, the entire 16-bit value is written, allowing for a custom font.

### Cursor
The cursor, when visible, always appears over the glyph (logical OR).

### ASCII
The standard glyph font represents visible ASCII characters from 0x20 to 0x7F, glyphs from 0x00-0x1F are supplemental graphic characters. The display does not follow the concept of ASCII control characters.

----

Copyright 2016 Meisaka Yukara (CC BY 4.0)

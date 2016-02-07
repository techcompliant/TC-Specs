Embedded Display Controller
----

```
ASCII LOGO
```

|     Item       |   Value    |   Comment
| -------------: | ---------- | ----------------
|    Vendor code | 0x59EA5742 | Meisaka Engineering and Integration
|      Device ID | 0x70E3E4FF | EDC
|    Device type | 0x70E3     | Text cell based, command driven, monochrome display
|        Version | -          | Determines display characteristics

Embedded small LCD display controller.

Interrupt Commands
----

- **0x0000**: DISPLAY_CONTROL
    - B: Display options depend on each individual bit.
    ```
    ...00000
       |||||
       ||||Display power: 0 is off, 1 is on
       |||Backlight power: 0 is off, 1 is on (ignored if no backlight present)
       ||Cursor display: 0 is off, 1 is on
       |Cursor blink: 0 is steady, 1 is blink
       Cursor style: 0 is block, 1 is underscore

    Other bits are ignored by the display.
    ```
- **0x0001**: CURSOR_ADDRESS
    - Sets the cursor address to B.
        - Line 0 always starts at 0x0000;
        - Line 1 always starts at 0x0080;
        - Line 2 always starts at 0x0100;
        - Line 3 always starts at 0x0180;
        - Glyph RAM starts at 0x0200 and ends at 0x0400;
        - Values above 0x03FF are ignored and do not affect the display;
        - Values beyond line length will put the cursor offscreen, the next character written will not be visible, and will reset the cursor to the beggining of the next line;
        - When the cursor is set to glyph RAM, it will count up each write until it reaches the end of glyph RAM at 0x0400, at that point it will return to 0x0000.
- **0x0002**: RESET_CONTROL
    - B controls what action to take:
        - Bit 0: Clear screen if set;
        - Bit 1: Home cursor to address 0 if set.
- **0x0003**: WRITE_DISPLAY
    - B controls what to display:
        - Bits 0-7: Character to display:
            - 0x00-0x7F: built-in font glyphs;
            - 0x80-0xFF: character RAM glyphs.
        - Bit 8: Don't advance cursor (overwrite);
        - When cursor is in glyph RAM, writes the entire 16 bit value and advances.
- **0xFFFE**: QUERY_DISPLAY
    - For serial connections: Causes the display to transmit 5 words containing vendor code, device ID and version;
    - For direct DCPU connections: Has the same effect as an HWQ instruction.
- **0xFFFF**: RESTART-DISPLAY
    - Resets all display settings and zeroes RAM.


Behaviours
----

### Glyph RAM
* The glyph RAM holds up to 128 custom raster glyphs; each glyph is 4 words;
* Glyphs are rastered by the display horizontally, 8 bits per cell row;
* On  the standard 5x8 cell this is:
    * Word 0: Bits 0-4 are row 1, columns 1-5;
    * Word 0: Bits 8-12 are row 2, columns 1-5;
    * Word 1: Bits 0-4 are row 3, columns 1-5;
    * Word 1: Bits 8-12 are row 4, columns 1-5;
    * Word 2: Bits 0-4 are row 5, columns 1-5;
    * Word 2: Bits 8-12 are row 6, columns 1-5;
    * Word 3: Bits 0-4 are row 7, columns 1-5;
    * Word 3: Bits 8-12 are row 8, columns 1-5.
* When a WRITE_DISPLAY query is called while the cursor is in the glyph RAM, the entire 16-bit value is written, allowing for a custom font.

### Cursor
The cursor, when visible, always appears over the glyph (logical OR).

### ASCII
The standard glyph font represents visible ASCII characters from 0x20 to 0x7F, glyphs from 0x00-0x1F are supplemental graphic characters. The display does not follow the concept of ASCII control characters.

----

Copyright 2016 Meisaka Yukara (CC BY 4.0)

Interlaced Monochrome Videographic Adaptor
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
|      Device ID | 0x75F6A113 | IMVA
|    Device type | 0x75F6     | Memory mapped monochrome display
|        Version | 0x0538     |

The Interlaced Monochromatic Videographic Adaptor (the device), offers a unique and powerful ocean of monochrome pixels for all of your graphics needs.
At 320 by 200 pixels, the device can accommodate a wide array of graphical functions and display more information at once than ever thought possible. Yes, it only offers two colors, but you can customize them!

The device also includes a mini-compositor overlay to display a simple prompt or achieve other effects.

Interrupt Functionality
----
When the device is interrupted, the A register will be interpreted as a command

 - **0x0000**: Activate Display

   Register B should be set to the base memory address for the main display area.
   - The display requires 4000 words of memory.
   - Setting the base to zero will disable output and put the connected display in standby.

 - **0x0001**: Set overlay

   - Register B should be set to the base memory address for the overlay.
   - Register C should be set to the word offset for the overlay.
   - Setting B to zero disables overlay

 - **0x0002**: Set Effects

   - Register B adjusts display colors:
     - bits 0-3 are the red level, 4-7 are the green level, 8-11 are the blue level.
     - bits 12-15 adjusts contrast between on/off pixels (0 is most contrasting, 0xF is least contrasting).

   - Register C adjusts overlay effect
     - bits 0-3 adjust blink rate, in 10ths of a second. (zero means no blink)
     - bits 4-5 select the overlay pixel combine mode:
       - 00: OR mode - default
       - 01: XOR mode
       - 10: AND mode
       - 11: COPY mode (only overlay pixels are used)

Raster Function
----

The device uses each word for 2 lines and 8 pixel columns, while scanning left to right and top down.
This results in 40 columns across the display 8 pixels wide each.
Each word is divided into 8 bits for each line, the lower 8 bits are used for odd lines, and the higher 8 bits for even lines.
Bits are read high to low, left to right, meaning bit 7 of word 0, is at line 1 column 1.

*Layout Example*

```
       ----------------------------------------------------
       |      Word 0 bits      |      Word 1 bits      |
Line 1 | 7| 6| 5| 4| 3| 2| 1| 0| 7| 6| 5| 4| 3| 2| 1| 0| etc.
Line 2 |15|14|13|12|11|10| 9| 8|15|14|13|12|11|10| 9| 8|
       |-----------------------|-----------------------|---
       |      Word 40 bits     |      Word 41 bits     |
Line 3 | 7| 6| 5| 4| 3| 2| 1| 0| 7| 6| 5| 4| 3| 2| 1| 0| etc.
Line 4 |15|14|13|12|11|10| 9| 8|15|14|13|12|11|10| 9| 8|

```

Overlay Function
----

The overlay (if enabled) is a small segment rasterized similar to the main display, only it is 16x16 pixels, and additionally combined with the main display.
The overlay is set with the *Set Overlay* function, the offset passed with this interrupt function determines where the upper left of the overlay begins.
The remaining overlay addresses are computed by hardware, which treats the overlay like a 16x16 block of screen that is displayed over the main screen content.

- Note that the overlay only effects what is displayed, not the backing memory.
- If the overlay is set to the right most column, the 8 bits that would be at column 41 are not displayed.
- Likewise, Lines of the overlay that would go past the bottom of the display are not visible either.
- Setting the overlay offset past the visible display area hides it.
- The effects setting controls if the overlay blinks, and how it is combined with the main display.
- For blink effect: on and off times are equal.
- The offset function of the overlay, allows it to only be placed on odd lines.


Revision Notes
----

Version 5.25: Made the heat sink bigger - device no longer catches fire.
Version 5.10: Fix overlay corruption is some cases...
Version 5.00: Added 16x16 overlay.
Version 3.12: Customizable color, number 1 most demanded feature added.
Version 2.56: Added timing control for blink function.


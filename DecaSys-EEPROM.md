EEPROM
----
```
 ______    ______   
|_   _ `..' ____ \  
  | | `. | (___ \_| 
  | |  | |_.____`.  
 _| |_.' | \____) | 
|______.' \______.' 
                    
```
Device Description
----
The DecaSys EEPROM can be built into various devices to store supplementary data.  Holds 8 words, that can only be read or written a single word at a time.
Due to constraints in trying to build this capability into such a small package, bits in a single word can only be CLEARED.  In order to SET a bit, you have to reset the entire device to all 1's, and then set individual words to their new values.

Interrupt Commands
----
All interrupt commands are done with A set to 0xFFF0, sent to the same device as the EEPROM is attached to.  Register B is used to select the command, and X and Y are used for address and data.
 - A - **0xFFF0**:
   - B - **0x0001**: READ
     - Reads word from address X and stores it in register Y.
   - B - **0x0002**: WRITE
     - Writes word from register Y to address X.  Note that writing to the EEPROM can only clear bits, not set them, so you may need to clear the EEPROM to write new values.
   - B - **0x0003**: RESET
     - Resets all words to 0xFFFF
      

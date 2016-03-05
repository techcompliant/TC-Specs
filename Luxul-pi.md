Luxul Photon Indicator
----

```

                                 ^                                        
                                !!!                                       
                               !!!!! 
                           !!!!!!!!!!!!!                                  
                          !!!!!!! !!!!!!!                                 
                           !!!!  !  !!!!                                  
                            !   !!!   !                                   
                              !!!!!!!                                      
                              !!!!!!!                                     
                               !!!!!                                      
                                !!!                                       
                                 !          
                             L U X U L
                          
```

|     Item       |   Value    |   Comment
| -------------: | ---------- | ----------------
|    Vendor code | 0x010C337D | (LUXUL)
|      Device ID | 0x03       | (PI) 
|    Device type | 0x41F2     | LED, multi color, wired, indicator
|        Version | 0x2A       | Version Code

The Lulux Photon Indicator is a small command driven led. It is ideal for physical interfaces and giving notification to users. It is capable of displaying 16 different colors. Commands are given through KaiComm SSI.

Commands
----
 - **0x0000**: Query Device Info: Sends Vendor Code, Device ID, Device Type, and Version over KaiComm-SSI card.
 - **0x0001**: Get status: Sends current color status in the folowing bit format. 
 
 The second and third bits are ignored if power ≠ 2. If the most significant bit is set higher than 2, the device assumes the off state.
 
 
| Bit Significance | Most Significant     | Second Most Significant                      | Second Least Significant                      | Least Significant                                  |
|------------------|----------------------|----------------------------------------------|-----------------------------------------------|----------------------------------------------------|
| Range: 0x        | 0-2                  | 0-F                                          | 0-F                                           | 0-F                                                |
| Holds            | Power State          | Blink On                                     | Blink Off                                     | Color                                              |
|                  | 0=off, 1=on, 2=blink | How many decaseconds to stay on during blink | How many decaseconds to stay off during blink | Hex representation of color (Same spec as LEM1802) |
                                      
 - **0x0002**: Set status: Sets color status in the folowing bit format. 
 
 The second and third bits are ignored if power ≠ 2. If the most significant bit is set higher than 2, the device assumes the off state.
 
 
| Bit Significance | Most Significant     | Second Most Significant                      | Second Least Significant                      | Least Significant                                  |
|------------------|----------------------|----------------------------------------------|-----------------------------------------------|----------------------------------------------------|
| Range: 0x        | 0-2                  | 0-F                                          | 0-F                                           | 0-F                                                |
| Holds            | Power State          | Blink On                                     | Blink Off                                     | Color                                              |
|                  | 0=off, 1=on, 2=blink | How many decaseconds to stay on during blink | How many decaseconds to stay off during blink | Hex representation of color (Same spec as LEM1802) |
                                      
Examples
----  
` 0x0002 0x0FFF ` -
Turn pi off

` 0x0002 0x2AAF ` -
Blink white. one second on, one second off.

` 0x0002 0x1FF0 ` -
Turn on. Set color to black.

` 0x0000 ` - pi responds ` 0x010C 0x337D 0x0300 0x41F2  0x2A00 `


A note from luxul
----
Although this device is fully functional, it has been recalled. A faulty capacitor occationally discharges and can lead to combustion. If you own a luxul pi 2A, we advise you send it to the following address for a free replacement unit.
> Sector: C-137

> SHID: ZZ8-A1426 Berling Hall

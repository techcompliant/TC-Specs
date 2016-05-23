```
                              __
                 ____________ ____
             _______________  _________
          _________________  ______________
       ___________________  __________________                 __________________
     _____________________  ________________________        ________________________
    _______________________  ______________    _____          _______      __________
   _________________________  ________________     __    ________________    ________
  ___________________________ ____         _______      _______    ________   _______
  ____________________        _                ___        ________    _____    ______
  _________________                                        _________   _____   ______
  ________________                                         _________   _____   ______
  ________________                                       ___________   ____   ______
  ________________                                   ______________    ____  ______
  _________________                              _________________    ___   _____
   ____________________                    ______________________    __   ____
    __________________________________________________________
     _______________________________________________________
       __________________________________________________
         ____________________________________________
            ____________________________________
                 _______________________

-----------------------------------------------------------------------------------------
Hardware Info : 
-----------------------------------------------------------------------------------------
Device Name       : High Speed Data Printer Model 1Data (HSDP-1D)
Device ID         : TBD
Device Type       : TBD
Manufacturer Name : SlapDragon
Manufacturer ID   : 0xF6976D00

-----------------------------------------------------------------------------------------
Description :
-----------------------------------------------------------------------------------------
The HSDP line is all about printing as fast as possible the data you need.

The HSDP offer you the best way to store you data in term of cost efficency.
Paper power and data is all you need to run a HSDP, its clever design combining the 
speed of the line matrix and the lack of nedd for ink of a spark printer design will 
really cut the cost of your radio survey by orders of magnitude.

The HSDP line is also very quiet, SlapDragon guarantee less than 95dB of noise at full 
speed !

The HSDP-1D is a Data version of the model 1, equiped with curring edge memory chips 
and buses this printer will be able to log everything you throw at it.
It is built to print large amount of data in a smaller amount of time and lack 
typesettings possibilities of the Graph variant.

-----------------------------------------------------------------------------------------
Interrupt behavior :
-----------------------------------------------------------------------------------------
The HSDP line is built around modes, the model 1D has only two modes.

|ID | Mode Name | Line Content              | Line Size |
|---|-----------|---------------------------|-----------| 
| 0 | Text Mode | 80 ASCII characters/line  |         80|
| 1 | Data Mode | 16 words/line             |         16|


0 : Set Mode
    Reads register B and set the mode at B&0x1
	
1 : Get Mode
    Set DCPU register A to current mode
	
2 : Cut page
    Cuts the last printed page

3 : Print simple line
    Reads register B and prints a full line (according to current mode) from DCPU's 
	memory stating at B

3 : Print multiple lines
    Reads register B and C and prints C full lines (according to current mode) from 
	DCPU's memory stating at B
 
4 : Full Dump
    Reads and prints the full memory of the DCPU halting 


! WARNING !
Printing more than 2048 bytes of data may in 1 interupt result in halting your DCPU.
But be reassured that no printer of the HSDP line can suffer from any amount of data 
from a DCPU.

```

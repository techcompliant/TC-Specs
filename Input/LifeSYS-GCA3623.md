GCA3623 and GCA1823
----
```
  _____      ______   
 |_   _|   .' ____ \  
   | |     | (___ \_| 
   | |   _  _.____`.  
  _| |__/ || \____) | 
 |________| \______.' 
 LifeSys, a wholly owned subsidiary of DecaSys
```

|     Item       |   Value    |   Comment
| -------------: | ---------- | ----------------
|    Vendor code | 0x4c534453 | (LifeSys)
| DCPU Device ID | 0x2FF23233 | (LifeSys GCA)
|        Version | 0x1823     | GCA1823 PDA Module
|        Version | 0x3623     | GCA3623 Standalone Unit

Device Description
----
The LifeSys GCA3623 and GCA1823 are fully automated Gas Chromotography Analyzers, capable of analyzing any gas piped into them.
The GCAs use only a very minute amount of gas, and are capable of determining the exact nature of the gases, as well as their concentrations.
The GCA must be told to start an analysis operation, and thereafter takes approximately 10~20 seconds (15~30 seconds for the GCA1823) to fully analyze the gasses, depending on the number of gasses to analyze.  In addition, the GCA will need 10 seconds to flush and recover from the analysis.

Control
----
The GCA3623 and the GCA1823 use the same control protocol, though the GCA3623 runs over HIC and the GCA1823 runs off of interrupts from the DCPU.
For the GCA3623, the commands are passed via HIC, and needs 1 or 2 words, the first being the command number, and the second being the parameter, if required.
For the GCA1823, the commands use the HWI interrupt interface.  The command number should be in register A, and the parameter, if one is needed, should be in register B.

Commands
----
  - **0x0000**: GET DEVICE STATUS
    - Gets the devices current status.  2 words, the first being the status code, and the second being the number of gasses identified in the latest run.  Status Codes:
      - 0x0000: Device in standby, ready to perform an analysis.
      - 0x0001: Device performing an analysis.
      - 0x0002: Analysis complete, device flushing and recovery in progress.
      - 0xFFFF: Device malfunctioning.
  - **0x0001**: START ANALYSIS
    - Commands the device to begin an analysis.  Analysis time varies depending on multiple factors.
    - Once analysis has begun, prior results are no longer accessible.
  - **0x0002**: GET GAS
    - This command uses the second paramter to specify which gas index to get information on.  This parameter must be between 0 and the number of gasses minus 1.
    - Returns 2 words, the first being an identifier for the specific gas, and the second being it's relative concentration, between 1 and 65535.  All of the gasses in the gas index will add up to a total concentration of 65535.
    - Gas Identifiers:
      - 0x0001: Oxygen (O2)
      - 0x0002: Carbon Dioxide (CO2)
      - 0x0003: Nitrogen (N2)
  - **0x0003**: GET PRESSURE
    - This command returns the total pressure of the gasses at the start of the latest completed analysis cycle.
    - Return is in decapascals
  - **0xFFFE**: DEVICE QUERY
    - For serial connections: Causes the display to transmit 5 words containing vendor code, device ID and version
    - For direct DCPU connections: Has the same effect as an HWQ instruction.
  - **0xFFFF**: DEVICE RESET
    - Resets the device, removing the latest test results, and canceling any analysis in progress.  If an analysis is in progress, the device will need to flush and recover before being able to perform another analysis.

Speaker Hardware
----

```
No ASCII logo yet
```

|     Item       |   Value    |   Comment
| -------------: | ---------- | ----------------
|    Vendor code | 0x5672746B | VARTOK_HW
|      Device ID | 0x02060001 | Speaker
|    Device type | 0x0206     | Fixed-amplitude speaker
|        Version | 0x0001     | Version Code

This hardware can generate beeps on two separate channels with variable frequency. The amplitude is fixed for both of them with the second channel being slightly quieter.

Interrupt Commands
----

- **0x0000**: SET_FREQUENCY_CHANNEL_1
    Set frequency of first channel to value read from register B in Hz.

- **0x0000**: SET_FREQUENCY_CHANNEL_2
    Set frequency of first channel to value read from register B in Hz.

Behaviours
----
If the value in B is 0, that channel is turned off (which is the default state assumed at start-up).

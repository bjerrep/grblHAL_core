## grblHAL / core ##



This is a experimental fork of https://github.com/grblHAL/core intended to be used with the grblHAL STM32  driver [STM32H7xx](https://github.com/dresco/STM32H7xx) fork [here](https://github.com/bjerrep/STM32H7xx). The STM32H7xx fork (last link) is modified to take the submodule 'grbl' as this repository.

Please note that this repository and the STM32H7xx fork will almost per definition always be out-of-date and/or appear to be rather unmaintained.



### Experiment 1

Given enough controllers and SD cards floating around it really would be nice to tag the firmware so it can identify itself regarding the details and purpose of the build. 

Now the response to a $I will start with an additional build date and an info line:

```
[VER:1.1f.20241222:]
[BUILDDATE:Jan  2 2025 00:40:25]
[BUILDINFO:RELEASE:0x08020000:fix_541_with_warnings]
[OPT:VNMSL,100,1024,3,0]
....
```





### Experiment 2

This is what so far appears to be a fix for [grblHAL/core issue 547](https://github.com/grblHAL/core/issues/) seen when Freecad generates rather wide arcs that sends grblHAL in a trajectory to somewhere far away. The fix is actually just to remove a few lines. The problem was seen on a hard float STM32H743 (on a BTT SKR3 controller).


## grblHAL / core ##



This is a experimental fork of https://github.com/grblHAL/core intended to be used with the grblHAL STM32  driver [STM32H7xx](https://github.com/dresco/STM32H7xx) fork [here](https://github.com/bjerrep/STM32H7xx). The STM32H7xx fork (last link) is modified to take the submodule 'grbl' as this repository.

Please note that this repository and the STM32H7xx fork will per definition always be super out-of-date and/or appear to be rather unmaintained. This repository is most of all just a backup of the firmware for a BTT SKR3 controller that seems to work on at least my CNC. This fork contains a few socalled experiments:



### Experiment 1

Given enough controllers and SD cards floating around with different modified firmwares it can rather quickly be slightly challenging to keep track of what is running where. The existing version number is nice but with this modification the response to a $I will contain an additional build date and a free form text label:

```
[VER:1.1f.20241222:]
[BUILDDATE:Jan  2 2025 00:40:25]
[BUILDINFO:RELEASE:0x08020000:nozero3_atan2]
[OPT:VNMSL,100,1024,3,0]
....
```



### Build symbol

```
BUILD_MSG="nozero3_atan2"
```

### Code in report.c

```
    hal.stream.write(":");
    hal.stream.write(line);
    hal.stream.write("]" ASCII_EOL);

	✂
	
    // General build info. Define an optional BUILD_MSG symbol to add a custom text with e.g. firmware author or purpose
    hal.stream.write("[BUILDDATE:" __DATE__ " " __TIME__ "]" ASCII_EOL);
    hal.stream.write("[BUILDINFO:");
    #ifdef DEBUG
        hal.stream.write("DEBUG");
    #else
        hal.stream.write("RELEASE");
    #endif
    extern unsigned int g_pfnVectors;
    sprintf(buf, ":0x%08X:", (unsigned int) &g_pfnVectors);
    hal.stream.write(buf);
    #ifdef BUILD_MSG
        // Remember to define BUILD_MSG as a "string"
        hal.stream.write(BUILD_MSG);
    #else
        hal.stream.write("UNDEFINED");
    #endif
    hal.stream.write("]" ASCII_EOL);

	✂
	
#if COMPATIBILITY_LEVEL == 0
    extended = true;
#endif
```



### Experiment 2

This is what so far appears to be a fix for [grblHAL/core issue 547](https://github.com/grblHAL/core/issues/) seen when Freecad generates arcs that sends grblHAL in a trajectory to somewhere far far away. The problem was seen on a hard float STM32H743 (on a BTT SKR3 controller).

The thesis is that the original grbl code is trying to support both arcs and exact circles but implements it in a way that fails when Freecad suddenly asks for a small arc segment with a diameter in the kilometer range. The Freecad gcode might be unreasonable, but is valid nevertheless. And the CNC will end the job with a head crash or at least at a limit switch. So far the following code seems to work:



### Code - motion_control.c

        // Start of experimental fix for grblHAL core issue #547
    
        float angular_travel = 0.0;
    
        if ((rv.x - rt.x == 0.0) && (rv.y - rt.y == 0.0)) {
            // If we end up where we started then this is per definition a full circle covering 2*pi radian.
            angular_travel = 2.0f * M_PI;
        }
        else {
            // CCW angle between position and target from circle center. Only one atan2() trig computation required.
            angular_travel = (float)atan2(rv.x * rt.y - rv.y * rt.x, rv.x * rt.x + rv.y * rt.y);
            if (turns > 0)
            {
            	// G3 CCW
    			if (angular_travel < 0.0) {
    				angular_travel += 2.0f * M_PI;
    			}
            }
            else
            {
            	// G2 CW
    			if (angular_travel > 0.0) {
    				angular_travel -= 2.0f * M_PI;
    			}
            }
        }







## Experiment 3

Zeroing on a BTT SKR3 has a noticable delay which turns up to be due to a save to flash based non-volatile storage. Here the 913 is abused so if disabling 913 the save are skipped. This was just an investigation into why there was a slight blackout while zeroing.


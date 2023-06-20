# nanoGRBLizer
nanoGRBLizer is a GRBL 1.1 based Arduino Nano controller for Stepcraft SC420 D2 milling machine.
The design is based on the well known GRBLizer controller specifically designed for the Stepcraft mills by Rogier Lodewijks (see https://github.com/eflukx/Stepcraft-GRBLizer). 
The nanoGRBLizer controller uses the open source GRBL1.1h software (https://github.com/gnea/grbl/wiki) and allows in combination with a GRBL code sender (eg. UltimateCNC or UGS) controll of all 3 axes and the spindle of the stepcraft mill. A probe length sens input is also available. 

The functionality of the nanoGRBLizer compared to the original GRBLizer lacks the connections to controll a 4th axis and an additional relais. Dimensions and functionalty of the nanoGRBLizer PCB allow for a 1:1 replacement of the WinPCNC USB controller of the Stepcraft mills, which is often used for hobby purposes.

![schematic](nanoGRBLizer Schematic.pdf)

I first tried the original GRBLizer and ordered a PCB by PCBWay but realized that getting the required parts was problematic and costly. I therefore reduced the original design to the functionalities I really needed and had a specific focus on simplicity and cost. As a result I designed the nanoGRBLizer around a Arduino Nano bord which reduced the complexity a lot and was also very cost efficient. 

This repository holds the KiCAD files to replicate the nanoGRBLizer and gives hints for implementation, configuration and initial operation of the nanoGRBLizer.
## PCB Assembly
There are some caveeats which have to be respected during assembly of the PCB due to the space constraints of the Stepcraft mill.
- First the 6 pin ISP header of the Arduino Nano has to be shorted by about 3mm. Alternatively you have to unsolder the header or isolate it, as the metal shield of the Stepcraft driver board might shorten the pins.
- Directly solder the Arduino Nano on the PCB. Do not used a socket as the PCB might not fit into the available space and the USB socket might not be usable.
## Installation of GRBL1.1 firmware
- Download the current version of the GRBL 1.1 software from here: https://github.com/gnea/grbl
- Install the grbl lib according to your Arduino IDE
- Implement the changes to the files cpu_map.h and config.h as described by Rogier in https://github.com/eflukx/Stepcraft-GRBLizer
- Compile the grbl_uload.ino script from the grbl examples and download to your Nano
## Configuration of Controller
Implement the grbl settings as described by Rogier in his readme. They worked for me! Changed the minimum amd maximum speed of your spindle and the dimensions of your mill if required. 
## Initial Operation with UltimateCNC
I used UltimateCNC on Linux as a gcode sender and after initial connection to the nanoGRBLizer a error alarm showed up, which confused me a lot. After a few hours of debugging and head scratching I found the solution in the config.h file of the grbl library.

>/* If homing is enabled, homing init lock sets Grbl into an alarm state upon power up. This forces
 the user to perform the homing cycle (or override the locks) before doing anything else. This is
 mainly a safety feature to remind the user to home, since position is unknown to Grbl.
 */
>
>#define HOMING_INIT_LOCK // Comment to disable

Commenting the macro above and recompiling immediatelly solved this issue, but during Homing annother error "Alarmcode 8 - Homing Fail" showed up. The machine could be moved manually in each direction, but homing failed each time. Finnaly I found an cause for this error. The Stepcraft has all 3 end switches connected in series which are signaled to the Nano on pin 9. So if the x or y axes is in an endposition the homing will fail, as even if the z axis is clear from the endswitch x or y axis report endpositions. Just moving all axes manually clear of endpositions prior to homing solved this problem.

So if you try to use this controller I hope the information above will help you. Have fun!


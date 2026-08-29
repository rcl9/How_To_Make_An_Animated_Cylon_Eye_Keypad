# How to Make a Custom Numeric Keypad with an Animated 'Cylon Eye' + Z80 Source Code

This repository explains how I created the custom numeric keypad for my Phoenix (Z80) MIDI Computer and how the animated "Cylon Eye" (inspired from Battlestar Galatica) was implemented in hardware + Z80 assembly code. 

<div style="text-align:center">
<img src="/Images/Animated_CylonEye_500.gif" alt="" style="width:100%; height:auto;">
</div>

This is the Phoenix MIDI computer, its two main component boards, LCD display and keypad plus animated LED display:

<img src="/Images/128b.webp" alt="" style="width:75%; height:auto;"> 

## Wiring of the LEDs and Buttons:

The keypad is created from a 3 x 8 matrix of 0.5" push buttons:

<div style="text-align:center">
<img src="/Images/button_top2.webp" alt="" style="width:40%; height:auto;">       <img src="/Images/button_bottom.webp" alt="" style="width:40%; height:auto;">
</div>

for which the buttons and LEDs are connected to the computer via this ribbon cable assembly:

<img src="/Images/116b.webp" alt="" style="width:75; height:auto;"> 

## Custom Lettering of the Push Buttons

The crisp and custom lettering on each button looks like it was done with an inkjet printer but that was really not available at the time in 1985 to 1987 when I did this work. Rather, in the 1970s and 1980s graphics designers used *Letraset* of which I had wads of sheets to choose from in my collection. You can see the slight spacing issues on the F4 and F5 keys. These are the original sheets from which I stenciled on the white letters:

<img src="/Images/Letraset-1.webp" alt="" style="width:40%; height:auto;">  <img src="/Images/Letraset-2.webp" alt="" style="width:40%; height:auto;"> 

After the Letraset stenciling was complete I then applied several layers of Krylon protective spray. 40 years later it is still protecting the Letraset lettering quite well:

<img src="/Images/krylon.webp" alt="" style="width:40%; height:auto;">

## The 1ms Hardware Interrupt

The sequencing of the animated 'Cylon Eye' is designed around a 1ms interrupt provided via a 8253 timer and Z80 SIO interrupt.

Channel #3 of the Intel 8253 timer chip produces a "Timer interrupt" signal:

<img src="/Schematics/8253 timer.webp" alt="" style="width:75%; height:auto;"> 

which itself is sent to "/Sync A" (pin 11) on the Z80 SIO (serial I/O) chip as a hardware addessable non-maskable interrrupt:

<img src="/Schematics/Z80 SIO.webp" alt="" style="width:75%; height:auto;"> 

## Schematics & Software for the Keypad

The 3x8 keypad matrix is scanned via 3 bits sent out to the "Auxiliary Latch" and decoded from 3 bits to 8 bits via the 74LS138:

<img src="/Schematics/Keypad interface.webp" alt="" style="width:75%; height:auto;"> 

The return 3 status bits are read back via a Z80 PIO chip:

<img src="/Schematics/Z80 PIO.webp" alt="" style="width:75%; height:auto;"> 

This is the basic chip layout on the auxiliary board:

<img src="/Images/124b.webp" alt="" style="width:100%; height:auto;"> 

The file [Keypad scanner.mac](</Src/Keypad scanner.mac>) contains the Z80 assembly code to scan the keypad. The routine *scan$keypad* scans the keypad and maps the 3-bit value to a corresponding ASCII value via the [mapping tables](/Src/keytbls.mac). 

## Schematics of the LED Interface

The 8 LEDs are driven by a 74LS243 tri-state latch, current limited by a 180 ohm resister (leading to each LED being driven at 17mA).

<img src="/Schematics/LEDs driver.webp" alt="" style="width:75%; height:auto;"> 

## Software of the LED Interface

The Z80 assembly file [Cylon Eye.mac](</Src/Cylon Eye.mac>) contains the snippets of code to implement the animated *Cylon Eye* LEDs.

The *cylon$setup* code initializes the 8253 timer for a 1ms interval, the Z80 SIO for interrupts on its /SYNCA line and the initial LED #1 enabled. 

As the 8253 times-out it raises a Z80 non-maskable interrupt via the /SYNCA line on the SIO which in turn calls the rs232$ext$stat routine to animate the LEDs.


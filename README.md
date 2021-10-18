WARNING!
========
This repo is about to break for at least a couple of weeks while I integrate the STM32 hardware API
Currently I have a functional VGA display, and a functional USB keyboard(*with a known bug for device recognition).
If you liked the idea of basic running under Linux I'll keep backporting features into that anyway - see the new repo linuxretrobasic.
A consequence of this, is the code will become quite specific to the (very cheap) F767 development board from ST Micro, but easily ported to
any other NUCLEO as long as the MPU has an LCD module and you can solder some resistors to make an analog VGA interface....

RetroStyle STM32
==========
My primary goal is to transform robin edwards' great work into a standalone STM32 micro, "RetroStyle STM32". I've always wanted to recreate something that gave the 'instant' power up response like the 80's micros.

Current state
=============
make.sh script compiles code under 64bit Ubuntu it's fairly usable at the moment.
Looking at the code, the floating point handling might not be complete re. exponent formatted numbers.
Needs loads and save commands before it is really usable.

*  Terminal display:
Integrated an MIT licensed readline function***1** for line editor. Also clear screen and cursor positioning using vt terminal escapes.
Terminal IO tested using in gnome terminal. This should work using a serial terminal like screen too.
So it could be ported to other platforms at this point.
The readline module comes with a bunch of terminal utility functions. I need to check those out and maybe dump my vtxxx utilities.

* Interpreter:
Program entry, and most commands seem to work. [TODO, get some test written]
Fixed a 'bug' in integer value tokens, removing assumption that longs are 4 byte, which might be true in Arduino but wasn't in my env.

Added an EDIT command to BASIC so edit <n> will preload the line editor with an existing line of code
and enable changes without retyping a whole line.

Added support for E format floats, ie. 1e3 and 1e-3. All constants in exponent format are handled internally as, C floats.


Stages
------
1: convert files to .c, because that's what they are.

2: gently strip out or add preprocessor selections around all the Arduino specific code. I fully expect to break arduino compatability at some point but I'll try to keep changes to the basic interpreter clean enough to reintegrate back into the arduino interpreter unless they are very device specific.

3: build in a framework to allow compilation to Linux first because I've got a load of extensions to add to the BASIC language and most can be more easily tested outside the microncontroller device.

4: Remove dependencies on third party and GPL libraries, and trim anything that's more complex than needed for the destination microcontroller/cpu environment. This will set the final API for specifc host
cpu support.

5: start integrating the essential external peripherals - Keyboard - Display


Footnotes and credits
---------------------
*1 Credit to JC Wang: github:jcwangxp/Crossline for a very nice MIT release of a line editor
*2 Credit to Robin Edwards for his MIT release of Arduino BASIC.

--------------
BASIC Language reference
--------------
Variables names can be up to 8 alphanumeric characters but start with a letter e.g. a, bob32
String variable names must end in $ e.g. a$, bob32$
Case is ignored (for all identifiers). BOB32 is the same as Bob32. print is the same as PRINT

Types
```
Numeric variables are the same as the native compiled *float* format.
Constant numerical values are promoted to float, so 128 is 128.0.
Valid numbers include 2  2.2   2e3   2e-3  2E3  2E-3
  
Array variables are independent from normal variables. So you can use both:
```
LET a = 5
DIM a(10)
```
There is no ambiguity, since a on its own refers to the simple variable 'a', and a(n) referes to an element of the 'a' array.

```
Arithmetic operators: + - * / MOD
Comparisons: <> (not equals) > >= < <=
Logical: AND, OR, NOT
```

Expressions can be numerical e.g. 5*(3+2), or string "Hello "+"world".
Only the addition operator is supported on strings (plus the functions below).

Commands
```
PRINT <expr>;<expr>... e.g. PRINT "A=";a
LET variable = <expr> e.g. LET A$="Hello".
variable = <expr> e.g. A=5
LIST [start],[end] e.g. LIST or LIST 10,100
RUN [lineNumber]
GOTO lineNumber
REM <comment> e.g. REM ** My Program ***
STOP
CONT (continue from a STOP)
INPUT variable e.g. INPUT a$ or INPUT a(5,3)
IF <expr> THEN cmd e.g. IF a>10 THEN a = 0: GOTO 20
FOR variable = start TO end STEP step
NEXT variable
NEW
GOSUB lineNumber
RETURN
DIM variable(n1,n2...)
CLS
PAUSE milliseconds
POSITION x,y sets the cursor
PIN pinNum, value (0 = low, non-zero = high)
PINMODE pinNum, mode ( 0 = input, 1 = output)
LOAD (from internal EEPROM)
SAVE (to internal EEPROM) e.g. use SAVE + to set auto-run on boot flag
LOAD "filename", SAVE "filename, DIR, DELETE "filename" if using with external EEPROM.
```

"Pseudo-identifiers"
```
INKEY$ - returns (and eats) the last key pressed buffer (non-blocking). e.g. PRINT INKEY$
RND - random number betweeen 0 and 1. e.g. LET a = RND
GETCHAR$ - pauses program, waiting for a keypress, which is returned as a string
```

Functions
```
LEN(string) e.g. PRINT LEN("Hello") -> 5
VAL(string) e.g. PRINT VAL("1+2")
INT(number) e.g. INT(1.5)-> 1
STR$(number) e.g. STR$(2) -> "2"
LEFT$(string,n)
RIGHT$(string,n)

MID$(string,start,n)
PINREAD(pin) - see Arduino digitalRead()
ANALOGRD(pin) - see Arduino analogRead()

SIN(x) : x is any angle in radians, result is the sine of angle, between +/-1.0
COS(x) : x is any angle in radians, result is the cosine of angle, between +/-1.0

```


Original README (untouched) follows
===================================

Arduino Basic
=============
Now you can turn your Arduino into an 80's home computer!

A complete BASIC interpreter for the Arduino, using a PS/2 keyboard, and SPI OLED screen. The BASIC supports almost all the usual features, with float and string variables, multi-dimensional arrays, FOR-NEXT, GOSUB-RETURN, etc. Saving and Loading from EEPROM is supported, as well as auto-running a program on power-up. You can also read and write from the analog and digital pins.

There's about 1k of RAM available for your BASIC programs and variables, so its roughly equivalent to my first computer (a Sinclair ZX81). The other 1k of RAM (on an UNO) is used for the keyboard and screen buffers, with a small bit of room left for the CPU stack. That works out quite well, since there's a 1k EEPROM on the arduino so if your program fits in the basic environment, it will fit when saved to EEPROM!

[![Demo](http://img.youtube.com/vi/JB5RXoO1IwQ/0.jpg)](http://www.youtube.com/watch?v=JB5RXoO1IwQ)

Prerequisites
-------------
1: An Arduino with at least 2k of RAM. i.e. UNO/Duemilanove/etc. It should also work on the MEGA but is untested. I think the sketch is too big for the Leonardo, since it looks like the bootloader uses more RAM on this model.

2: A PS/2 Keyboard. See http://playground.arduino.cc/Main/PS2Keyboard for wiring details.

3: An SSD1306 based OLED Screen connected using SPI. See e.g. http://www.adafruit.com/product/938. I think it should also work fine with the 128x32 version - you'll just need to change a couple of defines.

4: (Optional) A Piezoelectric buzzer for keyboard clicks and other sounds.

5: (Optional) A external EEPROM (e.g. 24LC256) lets you save more than one file. You can pick these up for about £2/$2 on ebay.

Getting Started
---------------
1: Download the zip file, unpack and copy the *folder* to your arduino sketches directory.

2: Install the PS/2 keyboard library if required. I've included the version I used in the zip file.

3: Install the SSD1306ASCII library. The normal Adafruit library is too RAM hungry for this project, so I'm using a massively cut down driver instead. I've modified this library a bit to get fast SPI transfers which improved the screen updating speed by a factor of about 4. The modified version is included in the zip file.

For both libraries, unzip the files and copy the *folder* to your arduino libraries directory.

4: Check your wiring corresponds to the pins in the comments/defines at the top of the Arduino_BASIC file.



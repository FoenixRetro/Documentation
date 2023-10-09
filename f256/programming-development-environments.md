# Development Environments
Programs are usually developed by compiling or assembling the program on a regular PC or Mac. Most tools support all common platforms, including Linux, or can relatively easily be made to run using a program such as Bottles (Linux) or WineBottler (Mac). C compilers are available, and plenty of assemblers can also be used, possibly in combination with C.

Common editors include Visual Studio Code, Emacs, Vim or Visual Studio. Programs can be tested by either using FoenixIDE (an emulator) or the real hardware, and developers often use a combination of both. This document does not cover how to use the emulator, but its documentation can be found at https://github.com/Trinity-11/FoenixIDE.

# SuperBASIC
SuperBASIC is an excellent BASIC dialect, and can be used to write programs directly on the hardware. This is a great option that lets you experience the machine in all its glory. Using the emulator is of course also possible, and can be a good alternative when you don't have the machine with you.

For information on how to program SuperBASIC, please refer to [the reference manual](https://github.com/FoenixRetro/f256-superbasic/blob/main/reference/source/f256jr_basic_ref.pdf).

# C compilers
Compilers that can be used write a program in C are:

* Calypsi - https://www.calypsi.cc/
* WDCTools - https://wdc65xx.com/WDCTools
* CC65 - https://github.com/cc65/cc65

CC65 only supports the 65816 in emulation mode.

# Assemblers
Some available assemblers include:

* Calypsi - https://www.calypsi.cc/
* WDCTools - https://wdc65xx.com/WDCTools
* tass64 - https://sourceforge.net/projects/tass64/
* merlin32 - https://github.com/lroathe/merlin32
* ASMotor - https://github.com/asmotor/asmotor
* CA65 from the CC65 package - https://github.com/cc65/cc65

CA65 only has limited 65816 support.

# Emulators

There are two emulators available for the F256:

- [FoenixIDE](https://github.com/Trinity-11/FoenixIDE) includes an emulator and it's quite full featured.  You have dissembly and memory windows, and you can load PGZ files etc. into the emulator.   Windows only!

- [Junior Emulator](https://github.com/FoenixRetro/junior-emulator) uses [SDL](https://www.libsdl.org/) and hence can be made to work on Windows, Mac, and Linux.  While not quite a full featured as FoenixIDE, it works well.


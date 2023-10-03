# Starting F256 Programming Guide
This guide is meant as a starting point for programmers wanting to develop for the F256 computers. It is an overview of how the machine and its software environment is put together, something that can be difficult to otherwise piece together. It is not a kernel or SuperBASIC manual, nor is a technical hardware manual, these can be found elsewhere and studied at your leisure.

* [Hardware documentation](https://raw.githubusercontent.com/pweingar/C256jrManual/main/tex/f256jr_ref.pdf)
* [Kernel programming](https://github.com/ghackwrench/F256_MicroKernel/tree/master/docs)
* [SuperBASIC reference](https://github.com/FoenixRetro/f256-superbasic/blob/main/source/release/documents/f256jr_basic_ref.pdf)

# Introduction
Programming the F256 computers can be done in several ways, and all are both challenging and, more importantly, lots of fun.

The first option many will probably turn to is the flash resident SuperBASIC, the environment you see immediately after turning on the computer. This is a very powerful and fast BASIC, giving you access to most hardware features, and it also features inline assembly if you need that little extra speed. It is an excellent starting point.

SuperBASIC is backed by the MicroKernel, which is responsible for handling the different kinds of keyboards, file access and networking. You can create programs that communicate directly with the kernel and don't require SuperBASIC. This a very good option for games and programs that need to receive input from the user and access storage, but does not use any BASIC features. The kernel does not handle text and graphics output, this is something the program will have to implement itself.

Programs that use the kernel are started by the user, either from DOS or from SuperBASIC. The programs can be loaded from disk, or they can be resident in flash memory or RAM.

Lastly, you can of course go full bare metal. It's your computer, you have full control. After your program has been started by the user, you can simply opt to not use the kernel and handle everything yourself. You can also upload binary programs from a PC or Mac directly to memory, and have the RESET vector point to your entry point for the ultimate bare metal experience. Or you can completely replace the kernel with your own operating system. Roads? Where we're going we don't need roads.

# A Note on CPU
An F256 usually comes with a 65C02 CPU, but may optionally use a 65C816 or the FNX6809 (Motorola 6809 compatible). This document covers the F256 when configured with a 65C02 or 65C816.

# Memory Configuration
The F256s come with 512 KiB of RAM and 512 KiB of flash storage on-board. This memory is divided into 8 KiB banks. RAM is assigned bank number $00-$3F, and flash is assigned $40-$7F.

The machines have a Memory Management Unit (*MMU*), which is used to map this physical address space into 64 KiB which the CPU can then access. This is done by the use of a Lookup Table (or *LUT*), of which the MMU has four. A LUT is organized into eight 8 KiB *slots*, each pointing to a specific memory bank. One LUT is active at a time, and it is possible to edit another LUT without disturbing the one that is active.

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

# Example Programs
*Link to some simple "Hello, World!" type projects demonstrating how to set up and compile a program.*

# Distributing Programs
Programs should be distributed using one of the executable formats supported by the firmware. While it is possible to distribute a program as a binary blob, this requires users to tether the machine to a PC or Mac, and upload the program with a stand-alone tool. This is not very user-friendly and is discouraged. Binary files should only be used while developing a program.

# Firmware Structure
Several independent components make up the firmware, the most important one being the MicroKernel. The firmware includes other components that the kernel refers to as "kernel user programs" (or *KUP* for short). The kernel is responsible for switching between these when requested. These programs are currently the SuperBASIC environment, the SuperBASIC help viewer, a simple DOS commandline shell, and pexec. Further programs may be added in the future, and you are free to put whichever programs you may desire in a free slot.

The kernel is *only* able to start a user program from memory - flash or RAM.

These KUPs also have a name which is used to refer to them by users or code. They are:

|Program|Name|
|---|---|
|SuperBASIC|basic|
|SuperBASIC Help|help|
|DOS|dos|
|pexec| - |

SuperBASIC can switch to another KUP by issuing the "slash" command, eg. `/dos` will execute the KUP named `dos`. 

DOS can switch to another program just be typing its name on the commandline, eg `basic` to enter the SuperBASIC environment. DOS also implements its own functionality that enables loading KUPs from disk, this is not handled by the kernel.

`pexec` is the program responsible for loading and executing the *PGX* and *PGZ* file formats. In DOS this is done by issuing the command `- program.pgz`. This will execute the `pexec` program (remember, its commandline name is actually `-`), which will load and execute the commandline parameter, in this case `program.pgz`. As previously mentioned, SuperBASIC has the ability to execute a KUP, so from SuperBASIC, the equivalent command would be `/- program.pgz`.

Your code is able to switch to another program by calling the `RunNamed` kernel function.

# Co-existing with the Kernel
Many programs will have to interact with the kernel in some way. The kernel provides several useful services that take a great deal of work to implement, such as file handling, keyboard differences (remember, there are three different types of keyboard on the F256s!), and so forth.

The kernel reserves some resources for itself. This is MMU LUT #0 and #1, and RAM banks 6 and 7. Additionally, zero page locations $F0-$FF are reserved. You must therefore stay away from these completely.

A program is mapped into memory using LUT #3, and LUT #3 will be active when the program is handed control. This LUT will be configured to both contain RAM banks and the program (which may be flash or RAM). The last two slots are used by the kernel, and contains important working memory, entry points and interrupt vectors. These slots should not be modified, and they should also be copied to LUT #2, if you intend to switch to that. What slots 0-5 are configured as, depends on how and from where the program was loaded. This is covered in the "file format" section.

Documentation on how to use the kernel, and the facilities it provides, can be found in the [kernel repository](https://github.com/ghackwrench/F256_MicroKernel/tree/master/docs)

Of course, if you don't need the kernel, you can turn off interrupts, set up LUT 0 the way you prefer, and then switch to it. Although, you'll have to make sure you're not pulling the rug out from under you.

# File Formats
The F256 line has three file formats, each with slightly different properties and use cases. Additionally, pure binary files are often used by developers. The machine will be in a slightly different state, depending on how the program was started. We will cover this in the following sections.

## Binary
The binary file format is not a format as such. It is an image that is loaded directly into the F256's memory by a utility program running on the PC or Mac, and the F256 is then reset. Such an image must include properly initialized interrupt vectors, especially the RESET vector.

Running such an image requires knowledge of its starting address and a properly configured USB connection between the PC or Mac and the F256. Also, a jumper on the F256 Jr. motherboard must be moved to the "Boot-to-RAM" position. This is not necessary on the F256K.

Binary images should only be used by the developer while developing, as it is a very user-unfriendly method of distributing software, requiring tethering to a PC or Mac.

Please be aware that when doing this kind of bare metal coding, special care must be taken when the system has a 65C816 CPU installed. PIN 3 changed function between the two CPUs, and the hardware must be instructed to treat it accordingly, which should be performed as the first thing in the RESET entry point.

This requires a special coding sequence, but note that this is unnecessary when the machine was started under the kernel's control, the kernel has already done this for you. It only applies to uploading binary code images.

```
reset:
    ; ...

    ; If this is a 65816, switch pin 3 from an input
    ; (6502 PHI1-out) to a 1 output (816 ABORTB-in).

    ; Try to put the CPU into 65816 native mode.
        .cpu    "65816"
        clc
        xce             ; NOP on a w65c02

    ; Carry still clear if this is a w65c02;
    ; Carry set if an 816 was in emulation mode
    ; (as it would be after a RESET).
        bcc     +

    ; Switch back to emulation mode.
        sec
        xce

    ; Reconfigure CPU pin 3.
        stz     $1       ; io_ctrl
        lda     #3       ; 2 (DDR) + 1 (logic 1)
        sta     $d6b0

    ; Resume
+      .cpu    "w65c02"     
```

## KUP
KUPs are very simple and can contain 40 KiB code/data when run from flash, or 32 KiB when run from disk. Contained in their header is and entry point address, and the bank number of where they should be mapped into, the first possible bank being #1 ($2000).

The header is very simple:

```
Byte  0    signature: $F2
Byte  1    signature: $56
Byte  2    the size of program in 8k blocks
Byte  3    the starting slot of the program (ie where to map it)
Bytes 4-5  the start address of the program
Bytes 6-9  reserved
Bytes 10-  the zero-terminated name of the program.
```

When instructed to start a named KUP, the kernel will search through memory to find a match. First the expansion memory is searched, this could be either RAM or ROM. Then the on-board flash memory is searched. If DIP switch #1 is in the "on" position, the kernel will search memory banks 1-5 before the expansion blocks.

This sequence is also performed when the kernel starts up, for instance when the machine is reset or is first powered on.

Testing a KUP is therefore as easy as uploading it to a block of memory, and is an excellent choice for testing a program that interacts with the kernel. Though you should remember that this program will stay in memory as long as the machine is powered on, and may also survive a short power cut. Such a program should therefore, when testing, destroy its own header by overwriting the first two bytes with some values that are not the magic signature, to avoid being started again and again.

A KUP residing in flash does not "return" in the common sense of the word. If it has to terminate, it can either reset the machine, or use `RunNamed` to start another KUP.

On the other hand, when running a KUP from disk, it is able to return to DOS in a controlled manner. Distributing software intended to be run from disk as KUP is usually discouraged, as the program can only be started from DOS and not from SuperBASIC. It can, however, be appropriate for small utility programs mostly intended for maintenance tasks and so forth. When distributing software, you should prefer using one of the PGX or PGZ formats.

A KUP loaded from disk may exit back to DOS by issuing an RTS instruction. Additionally, the carry may be set if the machine should be reset, or carry clear to continue running DOS. Be aware that you must leave the MMU configuration in the state you found it for this to work properly. You should also keep away from the $0200-$07FF range completely.

When a KUP is started from flash, LUT #3 slots 0-5 is mapped to RAM banks 0-5, *except* where the KUP's header specifies where into should be mapped.

A KUP started from disk has LUT #3 slots 0-4 mapped to RAM, some of which will be the program itself.

In both cases the kernel is intact and available for use straight away.

## PGX
This format is a simple, single segment executable, which can be loaded as low as $0200. A PGX is loaded using `pexec`. `pexec` treats PGX and PGZ in exactly the same way, and information on how PGZ is handled also applies to PGX.

## PGZ
This is a more advanced multi-segment executable. It can be loaded as low as $0200. It is loaded using `pexec`.

When `pexec` loads a program, the program must not load itself into RAM banks 6 & 7, as these are used by the kernel, and the kernel is doing the actual loading. If the program does not need the kernel, it may of course utilize these banks once it is running. `pexec` is otherwise able to load code and data into anywhere in physical memory.

When the program starts, the state of the MMU LUTs is very similar to when a KUP starts. LUT #3 is active and slots 0-4 are mapped to RAM banks 0-4. The program's entry point must be in this region. Slots 6 and 7 are mapped to the kernel, which is intact and usable right away. Slot 5 is currently undefined.

Testing a PGZ requires copying it to a disk or SD card, and manually running it on the machine.

A PGZ cannot "return" to another program, it can either start a KUP (if the kernel is intact) or reset the machine.

# Startup Sequence
Assuming the machine is in "Boot-to-Flash"-mode (the usual mode), MMU LUT #0 will be active when it is powered on, RAM banks 0-6 mapped into slots 0-6, and the last block of flash memory mapped into slot 7, where the interrupt vectors (including RESET) also reside. The kernel is usually placed in the last block, which causes it to be the first program running.

When the kernel has initialized itself, it looks for the first program it should start. First the expansion memory is searched, this could be either RAM or ROM. Then the on-board flash memory is searched. If DIP switch #1 is in the "on" position, the kernel will search memory banks 1-5 before the expansion blocks. In most cases no programs are present in expansion memory or main memory, and the first program started is the first one in flash memory. This is usually SuperBASIC, although you can put anything you want there, possibly DOS.

# Parameter Passing
Although not part of the kernel specification, a standardized method of passing commandline arguments to programs exists.

Both DOS and SuperBASIC are able to pass arguments to the program to run, and `pexec` is also able to pass any further arguments after the filename on to the program. As an example, `/- program.pgz hello` in SuperBASIC would start `pexec` with the parameters `-`, `program.pgz`, and `hello`. `pexec` would then load `program.pgz`, and start it with the parameters `program.pgz` and `hello`.

Arguments are passed in the `ext` and `extlen` kernel arguments. This approach is suitable for passing arguments through the `RunNamed` and `RunBlock` kernel functions, and is also used by `pexec` when starting a PGZ or PGZ program.

`ext` will contain an array of pointers, one for each argument given on the commandline. The first pointer is the program name itself. The list is terminated with a null pointer. `extlen` contains the length in bytes of the array, less the null pointer. For instance, if two parameters are passed, `extlen` will be 4.

`pexec` reserves $200-$2FF for parameters - programs distributed in the PGX and PGZ formats should therefore load themselves no lower than $0300, if they want to access commandline parameters. If they do not use the commandline parameters, they may load themselves as low as $0200.

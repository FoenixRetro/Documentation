# Hardware and Firmware Overview

The [F256Jr](https://c256foenix.com/f256-jr/?v=7516fd43adaa) and the [F256K](https://c256foenix.com/f256k/?v=7516fd43adaa) are almost identical from a programming perspective.  The main differences are:
- 256K has OPL3 sound chip and the Jr doesn't.
- 256K's built in keyboard is implemented as an additional CBM-style (matrix) keyboard via a CIA whereas Jr's is PS/2 style.  The K also supports connecting a PS/2 external keyboard. The Jr also supports connecting a real CBM keyboard or electrically equivalent. The difference in keyboard interfaces is abstracted from you if you run on top of the MicroKernel, but if you take over the entire machine and run bare metal, you need to compensate for this difference.

# A Note on CPU
An F256 usually comes with a 65C02 CPU, but may optionally use a 65C816 or the FNX6809 (Motorola 6809 compatible). This document covers the F256 when configured with a 65C02 or 65C816.

# Memory Configuration
The F256s come with 512 KiB of RAM and 512 KiB of flash storage on-board. This memory is divided into 8 KiB banks. RAM is assigned bank number $00-$3F, and flash is assigned $40-$7F.

The F256 line supports an [optional memory expansion card](https://c256foenix.com/product/f256-256k-ram-cartridge/?v=7516fd43adaa) of 256 KiB.  If this is installed the additional RAM is mapped to banks $80-$9F.

The machines have a Memory Management Unit (*MMU*), which is used to map this physical address space into 64 KiB which the CPU can then access. This is done by the use of a Lookup Table (or *LUT*), of which the MMU has four. A LUT is organized into eight 8 KiB *slots*, each pointing to a specific memory bank. One LUT is active at a time, and it is possible to edit another LUT without disturbing the one that is active.

# Firmware Structure
Several independent components make up the firmware, the most important one being the MicroKernel. The firmware includes other components that the kernel refers to as "kernel user programs" (or *KUP* for short). The kernel is responsible for switching between these when requested. These programs are currently the SuperBASIC environment, the SuperBASIC help viewer, a simple DOS commandline shell, and pexec. Further programs may be added in the future, and you are free to put whichever programs you may desire in a free slot.

The kernel is *only* able to start a user program from memory - flash or RAM.

These KUPs also have a name which is used to refer to them by users or code. They are:

|Program|Name|
|---|---|
|xdev|Cross-development helper|
|SuperBASIC|basic|
|SuperBASIC Help|help|
|DOS|dos|
|pexec| - |

SuperBASIC can switch to another KUP by issuing the "slash" command, eg. `/dos` will execute the KUP named `dos`. 

DOS has several built-in commands for maintenance task, such as formatting disks, copying files and forth. DOS can switch to another program just by typing its name on the commandline, eg `basic` to enter the SuperBASIC environment. DOS also implements its own functionality that enables loading KUPs from disk, this is not handled by the kernel. The built-in commands are searched for a matching command name before the KUPs. In order to only search KUPs, and slash may be prepended to command name. This also means that a KUP can always be start with `/command`, no matter the environment. This is useful for providing users with instructions how to start your program.

`pexec` is the program responsible for loading and executing the *PGX* and *PGZ* file formats. In DOS this is done by issuing the command `- program.pgz`. This will execute the `pexec` program (remember, its commandline name is actually `-`), which will load and execute the commandline parameter, in this case `program.pgz`. As previously mentioned, SuperBASIC has the ability to execute a KUP, so from SuperBASIC, the equivalent command would be `/- program.pgz`.

Your code is able to switch to another program by calling the `RunNamed` kernel function.

# Startup Sequence
Assuming the machine is in "Boot-to-Flash"-mode (the usual mode), MMU LUT #0 will be active when it is powered on, RAM banks 0-6 mapped into slots 0-6, and the last block of flash memory mapped into slot 7, where the interrupt vectors (including RESET) also reside. The kernel is usually placed in the last block, which causes it to be the first program running.

When the kernel has initialized itself, it looks for the first program it should start. First the expansion memory is searched, this could be either RAM or ROM. Then the on-board flash memory is searched. If DIP switch #1 is in the "on" position, the kernel will search memory banks 1-5 before the expansion blocks. In most cases no programs are present in expansion memory or main memory, and the first program started is the first one in flash memory. In the recent versions of the firmware, this is the `xdev` program, but in older firmwares, this is SuperBASIC. You can put anything you want there, possibly DOS.

When `xdev` starts up, it looks for programs and files uploaded through FoenixMgr, and if found, either copies them to the file system or executes them. If nothing is found, it passing execution on to the next program in flash.


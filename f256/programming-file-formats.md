# File Formats
The F256 line has three file formats, each with slightly different properties and use cases. Additionally, pure binary files are often used by developers. The machine will be in a slightly different state, depending on how the program was started. We will cover this in the following sections.

# Binary
The binary file format is not a format as such. It is an image that is loaded directly into the F256's memory by a utility program running on the PC or Mac, and the F256 is then reset. Such an image must include properly initialized interrupt vectors, especially the RESET vector.

Running such an image requires knowledge of its starting address and a properly configured USB connection between the PC or Mac and the F256. Also, a jumper on the F256 Jr. motherboard must be moved to the "Boot-to-RAM" position. This is not necessary on the F256K.

Binary images should only be used by the developer while developing, as it is a very user-unfriendly method of distributing software, requiring tethering to a PC or Mac.

# KUP
KUPs are very simple and can contain 40 KiB code/data when run from flash, or 32 KiB when run from disk. Contained in their header is and entry point address, and the bank number of where they should be mapped into, the first possible bank being #1 ($2000).

The header is very simple:

```
Byte  0    signature: $F2
Byte  1    signature: $56
Byte  2    the size of program in 8k blocks
Byte  3    the starting slot of the program (ie where to map it)
Bytes 4-5  the start address of the program
Byte  6    header structure version
Bytes 7-9  reserved
Bytes ?-?  zero-terminated name of the program.
Bytes ?-?  zero-terminated string describing the arguments.
Bytes ?-?  zero-terminated string describing the program's function.
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

Testing a PGZ can be done by using FoenixMgr and the `xdev`requires copying it to a disk or SD card, and manually running it on the machine.

A PGZ cannot "return" to another program, it can either start a KUP (if the kernel is intact) or reset the machine.


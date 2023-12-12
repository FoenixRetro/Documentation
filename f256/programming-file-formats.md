# File Formats
The F256 line has three file formats, each with slightly different properties and use cases. Additionally, pure binary files are often used by developers. The machine will be in a slightly different state, depending on how the program was started. We will cover this in the following sections.

# Binary
The binary file format is not a format as such. It is an image that is loaded directly into the F256's memory by a utility program running on the PC or Mac, and the F256 is then reset. Such an image must include properly initialized interrupt vectors, especially the RESET vector.

Running such an image requires knowledge of its starting address and a properly configured USB connection between the PC or Mac and the F256. Also, a jumper on the F256 Jr. motherboard must be moved to the "Boot-to-RAM" position. This is not necessary on the F256K.

Binary images should only be used by the developer while developing, as it is a very user-unfriendly method of distributing software, requiring tethering to a PC or Mac.

# KUP
KUPs are very simple and can contain 40 KiB code/data when run from flash, or 32 KiB when run from disk. Contained in their header is and entry point address, and the slot number of where they should be mapped into, the first possible slot being #1 ($2000).

DOS expands on the KUP header defined by the MicroKernel in a backwards compatible way.

The header is very simple:

```
Byte  0    signature: $F2
Byte  1    signature: $56
Byte  2    the size of program in 8k blocks
Byte  3    the starting slot of the program (ie where to map it)
Bytes 4-5  the start address of the program
Byte  6    header structure version (indicates version of header, current 0 or 1)
Bytes 7-9  reserved
Bytes ?-?  zero-terminated name of the program.
Bytes ?-?  zero-terminated string describing the arguments. (in version >= 1)
Bytes ?-?  zero-terminated string describing the program's function. (in version >= 1)
```

The MicroKernel ignores the "argument" and "function" header entries. These are printed by DOS' `lsf` command, and only if the header version is greater or equal to 1.

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

PGZ is similar in scale to MS-DOS’s COM format, or the Commodore PRG format. It consists of a single segment of data to be loaded to
a specific address, where that address is also the starting address.

PGX starts with a header to identify the file and the starting address:

* The first three bytes are the ASCII codes for “PGX”.
* The fourth byte is the CPU and version identification byte. Bits 0 through 3 represent
the CPU code, and bits 4 through 7 represent the version of PGX supported. At the moment, there is just version 0. The CPU code can be 1 for the WDC65816, or 2 for the
M680x0.
* The next four bytes (that is, bytes 4 through 7) are the address of the destination, in big-endian format (most significant byte first). This address is both the address of the location in which to load the first byte of the data and is also the starting address for the file.

All bytes after the header are the contents of the file to be loaded into memory.

## PGZ
This is a more advanced multi-segment executable. It can be loaded as low as $0200. It is loaded using `pexec`.

When `pexec` loads a program, the program must not load itself into RAM banks 6 & 7, as these are used by the kernel, and the kernel is doing the actual loading. If the program does not need the kernel, it may of course utilize these banks once it is running. `pexec` is otherwise able to load code and data into anywhere in physical memory.

When the program starts, the state of the MMU LUTs is very similar to when a KUP starts. LUT #3 is active and slots 0-4 are mapped to RAM banks 0-4. The program's entry point must be in this region. Slots 6 and 7 are mapped to the kernel, which is intact and usable right away. Slot 5 is currently undefined.

Testing a PGZ can be done by using FoenixMgr and the `xdev` firmware component. It can of course also be copied to a disk or SD card, and manually run it on the machine.

A PGZ cannot "return" to another program, it can either start a KUP (if the kernel is intact) or reset the machine.

The first byte of the file is a file signature and also a version tag. If the first byte is an upper case Z, the file is a 24-bit PGZ file (i.e. all addresses and sizes specified in the file are 24-bits). If the file is a lower case Z, the file is a 32-bit PGZ file (all address and sizes are 32-bits in
length). Note that all addresses and sizes are in little endian format (that is, least significant
byte first).

After the initial byte, the remainder of the PGZ file consists of segments, one after the other. Each segment consists of two or three fields:

|Field|Size|Description|
|-|-|-|
|address|3 (`Z`) or 4 (`z`) bytes|The target address for this segment|
|size|3 (`Z`) or 4 (`z`) bytes|The number of bytes in the data field|
|data|`size` bytes|The data to be loaded (optional)|

For a particular segment, if the size field is 0, there will be no bytes in the data field, and the segment specifies the starting address of the entire program. At least one such segment must be present in the PGZ file for it to be executable. If more than one is present, the last one will
be the one used to specify the starting address.

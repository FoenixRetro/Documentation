# Co-existing with the Kernel
Many programs will have to interact with the kernel in some way. The kernel provides several useful services that take a great deal of work to implement, such as file handling, keyboard differences (remember, there are three different types of keyboard on the F256s!), and so forth.

The kernel reserves some resources for itself. This is MMU LUT #0 and #1, and RAM banks 6 and 7. Additionally, zero page locations $F0-$FF are reserved. You must therefore stay away from these completely.

A program is mapped into memory using LUT #3, and LUT #3 will be active when the program is handed control. This LUT will be configured to both contain RAM banks and the program (which may be flash or RAM). The last two slots are used by the kernel, and contains important working memory, entry points and interrupt vectors. These slots should not be modified, and they should also be copied to LUT #2, if you intend to switch to that. What slots 0-5 are configured as, depends on how and from where the program was loaded. This is covered in the "file format" section.

Documentation on how to use the kernel, and the facilities it provides, can be found in the [kernel repository](https://github.com/ghackwrench/F256_MicroKernel/tree/master/docs)

Of course, if you don't need the kernel, you can turn off interrupts, set up LUT 0 the way you prefer, and then switch to it. Although, you'll have to make sure you're not pulling the rug out from under you.

# Distributing Programs
Programs should be distributed using one of the executable formats supported by the firmware. While it is possible to distribute a program as a binary blob, this requires users to tether the machine to a PC or Mac, and upload the program with a stand-alone tool. This is not very user-friendly and is discouraged. Binary files should only be used while developing a program.

# Interacting with the Machine
When developing a program, you will frequently need to interact with the machine, in order to test your program. This can simply be done by copying your program to disk or SD card, moving the media to the machine, and running it from DOS.

Moving files between the machines quickly becomes cumbersome, and your may even wear out the SD card slot. The machines also feature a USB debug, which you can connect to a PC or Mac using a regular USB-A to mini USB cable. For Windows and Mac, a special driver is needed. A driver for Windows is included in the firmware package, while the Mac's driver only works only works on older versions of macOS.

To download a program through the USB connetion, the program [FoenixMgr]( https://github.com/FoenixRetro/FoenixMgr.git) is needed. FoenixMgr has several subcommands for flashing blocks, uploading binary images, and running user code in PGZ or PGX format.

# Parameter Passing
Although not part of the kernel specification, a standardized method of passing commandline arguments to programs exists.

Both DOS and SuperBASIC are able to pass arguments to the program to run, and `pexec` is also able to pass any further arguments after the filename on to the program. As an example, `/- program.pgz hello` in SuperBASIC would start `pexec` with the parameters `-`, `program.pgz`, and `hello`. `pexec` would then load `program.pgz`, and start it with the parameters `program.pgz` and `hello`.

Arguments are passed in the `ext` and `extlen` kernel arguments. This approach is suitable for passing arguments through the `RunNamed` and `RunBlock` kernel functions, and is also used by `pexec` when starting a PGX or PGZ program.

`ext` will contain an array of pointers, one for each argument given on the commandline. The first pointer is the program name itself. The list is terminated with a null pointer. `extlen` contains the length in bytes of the array, less the null pointer. For instance, if two parameters are passed, `extlen` will be 4.

`pexec` reserves $200-$2FF for parameters - programs distributed in the PGX and PGZ formats should therefore load themselves no lower than $0300, if they want to access commandline parameters. If they do not use the commandline parameters, they may load themselves as low as $0200.

# Binary images and 65C816
Be aware that when doing bare metal coding by uploading binary images directly to memory, special care must be taken when the system has a 65C816 CPU installed. PIN 3 changed function between the two CPUs, and the hardware must be instructed to treat it accordingly, which should be performed as the first thing in the RESET entry point.

This requires a special coding sequence, but note that this is unnecessary when the machine was started under the kernel's control, the kernel has already done this for you. It only applies to uploading binary code images.

The following code snippet will function on both 65C02 and 65C816, but will perform the additional initialization steps correctly.

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

# Example Programs
*Link to some simple "Hello, World!" type projects demonstrating how to set up and compile a program.*


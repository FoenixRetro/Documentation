# DOS
Features are being added to DOS to support loading of executables from SD card and disk.

This experimental DOS may be found on the [releases page](https://github.com/FoenixRetro/Documentation/releases).

This work is being done in [this GitHub repository and branch](https://github.com/csoren/F256_Jr_Kernel_DOS/tree/exe).

Presently loading [kernel user programs](https://github.com/ghackwrench/F256_Jr_Kernel_DOS/tree/main/kernel) are supported.

To load a program, place it on eg. the SD card, and run it from DOS by typing its name.

Arguments may be passed to the program by adding them after the program name. Included in the firmware download is a `hello` program. This may be placed on disk and run by typing `hello` at the DOS prompt. Additionally, arguments may be added and they will be printed by the program.

# Programming information
DOS uses the memory area $0000-$07FF. Parameters are also stored in this area, and they must be copied if the area is also used by the user program.

Kernel user programs comprise up to four banks of code, and may be placed in the area $2000-$9FFF. The program header provides information to DOS on where to load the program and its entry address.

The program will be called with DOS' MMU configuration as configured by the MicroKernel.

A program may exit back to DOS by issuing an RTS instruction. Additionally the carry may be set if the machine should be reset, or carry clear to continue running DOS. Be aware that you must leave the MMU configuration in the state you found it.

Arguments are passed in the `ext` and `extlen` kernel arguments. This approach has been cleared with Jessie Oberreuter, the MicroKernel author, and is also suitable for passing arguments through the RunNamed and RunBlock kernel functions.

`ext` will contain an array of pointers, on for each argument given on the commandline. The first pointer is the program name itself. The list is terminated with a null pointer. `extlen` contains the length in bytes of the array, less the null pointer. For instance, if two parameters are passed, `extlen` will be 4. These strings should be copied into the programs memory if it uses the area used by DOS for storage.

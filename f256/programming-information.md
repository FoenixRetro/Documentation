# Starting F256 Programming Guide
This guide is meant as a starting point for programmers wanting to develop for the F256 series computers. It is an overview of how the machine and its software environment is put together, something that can be difficult to otherwise piece together. It is not a kernel or SuperBASIC manual, nor is a technical hardware manual, these can be found elsewhere and studied at your leisure.

* [Hardware documentation](https://raw.githubusercontent.com/pweingar/C256jrManual/main/tex/f256jr_ref.pdf)
* [Kernel programming](https://github.com/ghackwrench/F256_MicroKernel/tree/master/docs)
* [SuperBASIC reference](https://github.com/FoenixRetro/f256-superbasic/blob/main/source/release/documents/f256jr_basic_ref.pdf)

# Table of Contents
The guide is divided into several sections:

* [Introduction](#introduction) to the F256 line
* [Hardware and System Overview](programming-hardware-overview.md)
* [Development Environments](programming-development-environments.md)
* [Executable File Formats](programming-file-formats.md)
* [Tips for developing](programming-developing.md)

# Introduction
Programming the F256 computers can be done in several ways, and all are challenging and, more importantly, lots of fun.

The first option many will probably turn to is the flash resident SuperBASIC, the environment you see immediately after turning on the computer. This is a very powerful and fast BASIC, giving you access to most hardware features, and it also features inline assembly if you need that little extra speed. It is an excellent starting point.

SuperBASIC is backed by the MicroKernel, which is responsible for handling the different kinds of keyboards, file access and networking. You can create programs that communicate directly with the kernel and don't require SuperBASIC. This a very good option for games and programs that need to receive input from the user and access storage, but does not use any BASIC features. The kernel does not handle text and graphics output, this is something the program will have to implement itself.

Programs that use the kernel are started by the user, either from DOS or from SuperBASIC. The programs can be loaded from disk, or they can be resident in flash memory or RAM.

Lastly, you can of course go full bare metal. It's your computer, you have full control. After your program has been started by the user, you can simply opt to not use the kernel and handle everything yourself. You can also upload binary programs from a PC or Mac directly to memory, and have the RESET vector point to your entry point for the ultimate bare metal experience. Or you can completely replace the kernel with your own operating system. Roads? Where we're going we don't need roads.


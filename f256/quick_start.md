# Getting programs onto the F256

## SD Card 

Both the F256K and F256JR have an SD card slot.  The device sofware to read the SD card is a bit touchy (it's inherited from the Commander X16 project) and doesn't work with all SD cards

- Card should be "XC" type vs. "HC".   Typically older cards in the 512MB-8GB range work pretty well, which is more than enough space anyways.  Very new cards that are very large tend not to work?
- Card *MUST* be formatted FAT32 -- **NOT: FAT, FAT12, FAT16, or exFAT**.   Note that while MacOS will read a FAT32 formatted card, the included disk utility won't format FAT32.  Windows 10/11 works fine, but make sure to force FAT32 (or use the command line: `format /FS:FAT32 H:`)
- Some folks have had luck formatting cards with the [official SD Association formatter for Windows](https://www.sdcard.org/downloads/formatter/sd-memory-card-formatter-for-windows-download/).


## Demos Archive

- [Download the most recent demos archive](archive/) and expand it to the root of the flash card.

## Debug USB port 

- This is what most developers use as it's the most convienent.  Connect the debug USB port to your PC or Mac

- You can use:
  - [FoenixMgr](https://github.com/pweingar/FoenixMgr) - works on Windows, Mac, Linux
     - A Python script to manage the Foenix series of retro style computers through their USB debug ports. This tool allows uploading files of various formats to system RAM, and displaying memory through various means.

  - [FoenixIDE](https://github.com/Trinity-11/FoenixIDE) (Windows only)
    - Development and Debugging Suite for the C256 Foenix Family of Computers.

# SuperBASIC

The machine boots to SuperBASIC.  SuperBASIC is inspired by BBC BASIC but offers quite a bit more.

- Read the [reference manual](https://github.com/FoenixRetro/f256-superbasic/blob/main/reference/source/f256jr_basic_ref.pdf).

- Watch EMWhite's [excellent intro series on Youtube](https://www.youtube.com/playlist?list=PLeHjTvk7NPiSqGz4REMH-S4hjYpLS2YNR).

To get started, you can type in a sample program at the command prompt:

```basic
10 for i=1 to 5
20 print "Hello world"
30 next
```

`dir` - Run this to display directory of SD card

Loading & running programs off of the SD card is similarly easy:

```basic
load "JrWordl.bas"
run
```

Read built-in help/reference:

`/help` : But NB, this erases BASIC memory!  Use Backspace key to go back in menus and to exit.

Explore the included demo SuperBASIC programs:

| Program | Notes | Source | 
| ------- | ----- | ------ |
| `JrWordl.bas` | Wordle game, guess 5 letter word
| `mandel.bas` | Draws Mandlebrot set in graphics mode, takes a few mins
| `Problematic_Code.bas` | Displays scrolling starfield
| `noelrl.bas` | Simple integer BASIC bench mark from Noel's retro lab.  Completes < 3.5 seconds, compares very favorably to other retro systems!| [Youtube](https://www.youtube.com/watch?v=H05hM_Guoqk)
| `dance.bas` | Animates sprite of dancer
| `luna.bas` | Displays simple scene
| `blink.bas` | Blinks drive access light


# Native Code

- Binary programs for the F256 line are typically distributed as `pgx` or `pgz` files (see [ref here](https://wiki.c256foenix.com/index.php?title=Executable_binary_file), they are like the `prg` format in the C64 ecosystem).

- SuperBASIC: Slash (`/`) command will execute the named flash resident program, such as `/dos` or `/help` (SuperBASIC reference).

- PGZ/X files can be run from SuperBASIC with `/- program.pgz`, and from DOS with `- program.pgz`

- The `-` is also referred to as `pexec`, and is a chainloader that understands `pgx` and `pgx`, so `/- program.pgz` first hands over control to `pexec` which then loads the program and hands over control.

- You will typically need to reset the machine to get back to SuperBASIC.

- Switch to DOS with `/dos` and back into BASIC once there with `basic`.  `help` display a list of available DOS commands.

Try the included native demo programs:

| Program | Notes | Source | 
| ------- | ----- | ------ |
| `balls.pgz` | Demonstrates 280 multiplexed sprites | [GitHub](https://github.com/FoenixRetro/demos/blob/main/README.md)


# More Resources

## Discord

The [Foenix Retro Systems Discord](https://discord.com/invite/aAEQXZHXgM) is the primary place to get questions answered

## C256Wiki

Explore all the content:
https://wiki.c256foenix.com/index.php?title=Special:AllPages

## Foenix Retro Systems Newletter

Read back issues [here](http://apps.emwhite.org/foenixmarketplace/) (also a great source for sample programs).   Issues starting at #4 cover the F256 line.  Issues 1-3 cover the previous version of the hardware (C256), although there are still many salient points.
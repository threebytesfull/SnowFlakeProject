# SnowFlakeFirmware - Command Line Build

This fork adds a Makefile for building the firmware with command-line tools.
I've only tested it on the Mac but it only needs the GNU ARM toolchain, GNU
make and a shell to build so should be portable.

# Prerequisites and Environment

## GNU Make

If you've already got development tools installed, you've probably got this
already (Xcode, Homebrew, etc).

## GNU ARM Toolchain

Download the GNU ARM toolchain from the official release page:

* [https://developer.arm.com/open-source/gnu-toolchain/gnu-rm/downloads](https://developer.arm.com/open-source/gnu-toolchain/gnu-rm/downloads)

I used the `6-2017-q2-update` version (archive rather than installer, so I can choose where to put it).

Unpack the toolchain archive:

    tar -C $HOME/ARM -xf gcc-arm-none-eabi-6-2017-q2-update-mac.tar.bz2

The GNU ARM toolchain needs to be in your PATH:

    export PATH="$HOME/ARM/gcc-arm-none-eabi-6-2017-q2-update/bin:$PATH"

## ARM CMSIS 5

Download the ARM CMSIS pack from the official releases page:

* [https://github.com/ARM-software/CMSIS_5/releases](https://github.com/ARM-software/CMSIS_5/releases)

I used [https://github.com/ARM-software/CMSIS_5/releases/download/5.1.1/ARM.CMSIS.5.1.1.pack](https://github.com/ARM-software/CMSIS_5/releases/download/5.1.1/ARM.CMSIS.5.1.1.pack).

Unpack the pack file with `unzip` (it's just a zipfile with a different extension):

    unzip -D $HOME/ARM/CMSIS-5.1.1 ARM.CMSIS.5.1.1.pack

The `CMSIS` environment variable should point to the `CMSIS` directory inside
it:

    export CMSIS="$HOME/ARM/CMSIS-5.1.1/CMSIS"

## Atmel SAMD20 Device Family Pack

Download the SAMD20 DFP from the official releases page:

* [http://packs.download.atmel.com](http://packs.download.atmel.com)

I used the version 1.2.91 pack.

The `SAMD20` environment variable should point to the `samd20` directory inside
it:

    export SAMD20="$HOME/ARM/SAMD20_DFP-1.2.91/samd20"

# Building

You can build the firmware with `make` inside the `SnowFlakeFirmware`
directory:

    cd SnowFlakeFirmware
    make

That will generate `main.bin`, `main.elf`, `main.hex` and `main.srec` - at
least one of those should be suitable for flashing, depending on the tools
you're using.

# Flashing

I've used both the Segger J-Link and the Black Magic Probe to upload code to
the Snowflake.

## Black Magic Probe

Flashing with the Black Magic Probe uses `gdb`:

    make flash-bmp

This script enables target power on the Black Magic Probe so you don't need an
external power supply for this to work.

## Segger J-Link

Assuming the Segger J-Link `JLinkExe` is in the path (you may have to change
this to `JLink.exe` on Windows?), you can flash with:

    make flash

Note that the J-Link doesn't provide power to the target board, so you'll need
to ensure your Snowflake board is powered up (and remember that it's a 3.3V
board!).

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

I used [https://github.com/ARM-software/CMSIS_5/releases/download/5.2/ARM.CMSIS.5.2.pack](https://github.com/ARM-software/CMSIS_5/releases/download/5.2/ARM.CMSIS.5.2.pack).

Unpack the pack file with `unzip` (it's just a zipfile with a different extension):

    unzip -D $HOME/ARM/CMSIS-5.2 ARM.CMSIS.5.2.pack

The `CMSIS` environment variable should point to the `CMSIS` directory inside
it:

    export CMSIS="$HOME/ARM/CMSIS-5.2/CMSIS"

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

To check that the J-Link can see your board, you can use the `jlink-status` target:

    make jlink-status

If the Snowflake board is connected and powered up, you should see something like this:

    JLinkExe -if swd -device ATSAMD20E17 -speed 15000 -CommanderScript segger-hwinfo.script
    SEGGER J-Link Commander V6.20f (Compiled Oct 13 2017 17:20:01)
    DLL version V6.20f, compiled Oct 13 2017 17:19:52


    Script file read successfully.
    Processing script file...

    J-Link connection not established yet but required for command.
    Connecting to J-Link via USB...O.K.
    Firmware: J-Link V10 compiled Oct  6 2017 16:37:55
    Hardware version: V10.10
    S/N: 260100021
    License(s): FlashBP, GDB
    OEM: SEGGER-EDU
    VTref = 2.966V
    HWInfo[00] = Target power is disabled
    HWInfo[02] = -12mA	(ITarget)
    HWInfo[03] = 13mA	(ITargetPeak)
    HWInfo[04] = 13mA	(ITargetPeakOperation)
    HWInfo[10] = 0ms	(ITargetMaxTime0)
    HWInfo[11] = 0ms	(ITargetMaxTime1)
    HWInfo[12] = 0ms	(ITargetMaxTime2)
    HWInfo[13] = 0x00000000

    VTarget=2.966V
    ITarget=-12mA
    TCK=1 TDI=0 TDO=0 TMS=1 TRES=1 TRST=0
    Supported target interface speeds:
    - 180 MHz/n, (n>=12). => 15000kHz, 13846kHz, 12857kHz, ...
    - Adaptive clocking

    Device "ATSAMD20E17" selected.


    Connecting to target via SWD
    InitTarget()
    Found SW-DP with ID 0x0BC11477
    Scanning AP map to find all available APs
    AP[1]: Stopped AP scan as end of AP map has been reached
    AP[0]: AHB-AP (IDR: 0x04770031)
    Iterating through AP map to find AHB-AP to use
    AP[0]: Core found
    AP[0]: AHB-AP ROM base: 0x41003000
    CPUID register: 0x410CC601. Implementer code: 0x41 (ARM)
    Found Cortex-M0 r0p1, Little endian.
    FPUnit: 4 code (BP) slots and 0 literal slots
    CoreSight components:
    ROMTbl[0] @ 41003000
    ROMTbl[0][0]: E00FF000, CID: B105100D, PID: 000BB4C0 ROM Table
    ROMTbl[1] @ E00FF000
    ROMTbl[1][0]: E000E000, CID: B105E00D, PID: 000BB008 SCS
    ROMTbl[1][1]: E0001000, CID: B105E00D, PID: 000BB00A DWT
    ROMTbl[1][2]: E0002000, CID: B105E00D, PID: 000BB00B FPB
    Cortex-M0 identified.


    Script processing completed.


For convenience, you can also get to the `JLinkExe` prompt with the chip and
connection details already set up:

    make jlink

Type `connect` to connect to the target board or `q` to quit. `?` will list
other options but see the manual for more details - that's beyond the scope of
this document. :)

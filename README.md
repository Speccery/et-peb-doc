# ET-PEB manual for Ciro

Project discussed at https://atariage.com/forums/topic/274150-eriks-et-peb/

# Introduction

The ET-PEB is a dual function extension device for the TI-99/4A. 
- Memory expansion
- DSR Support
- SD card based disk support

The ET-PEB hardware design is straightforward. There is a Xilinx CPLD chip, which connects to the TI-99/4A side connector as well to most other chips on the board.

A static RAM chip provides memory expansion and storage for DSR routines.

## Microcontroller
A microcontroller (currently NXP's LPC1347, but previously also LPC11U37 and LPC1343 have been used) implements the disk support using a SD card as the storage medium. The microcontroller connects to the CPLD, to the SD card and to a serial flash chip. The microcontroller implements FAT file system support for the SD card and serves the TI-99/4A for disk operations. It also loads the DSR routines so that the TI-99/4A has the necessary drivers to support disks.

## Memory expansion
The memory expansion supports the standard 32K RAM expansion as well as paged memory in the SAMS mode.

The current version of the board was designed for a 512K RAM chip, which provides 256K of SAMS compatible paged memory expansion. With a couple of jumper wires and a revised CPLD design a 2MB RAM chip can be used, providing support for full 1MB of SAMS memory.

## DSR Support
In addition to the RAM memory expansion, the ET-PEB also supports DSR ROM support, in the default CRU address of >1100. 

# SD-card contents

## DSR file location
The TI-99/4A needs a DSR (Device Service Routine) to be able to support disk access. In the case of the ET-PEB the DSR is a very simple program, which communicats with the onboard microcontroller and asks it to do the operations.

Note that currently an SD-card inserted into the ET-PEB must contain the DSR file for the TI-99/4A. This is stupid and will be changed, since the ET-PEB board has flash and EEPROM space where software can be stored. But currently the root directory of the SD card needs to contain the file **ETDSR.BIN** which will be loaded on bootup of the ET-PEB. Thus the SD card must be in the SD card slot with the file for things to work.

## Disks
The ET-PEB uses a very simple FIAD scheme (files on disk) to store files. The default mount points are:
- Directory DSK1 for drive DSK1
- Directory DSK2 for drive DSK2
- Directory DSK3 for drive DSK3
These can be changed using serial connection (serial over USB) to the ET-PEB and the mount command there. Note that any changes to the mount points are not saved (yes it's stupid).

# Pin headers 

## MCU PROG
This is the most important pin header. It has 6 pins:
* Pin 1 is PIO1_4 and can be ignored.
* Pin 2 is RXD, serial port receive. This is a 3.3V signal. Can be normally ignored.
* Pin 3 is TXD, serial port transmit. By default it transmits MIDI output. 3.3V signal level.
* Pin 4 is LPCBOOT, boot mode selector. By default this signal is pulled high to 3.3 volts. When you want to enter firmware update mode using one of the methods below. When in firmware update mode, the ET-PEB appears as a tiny disk drive over USB. Firmware image can be dragged and dropped to this drive.
  - Pull this to ground with a jumper wire and reset the MCU or
  - Connect to ground with a jumper wire and then power on the ET-PEB
* Pin 5 is LPCRESET, reset signal of the microcontroller. This is normally pulled high to 3.3 volts. By briefly shorting it to ground (pin 6) the MCU can be reset.
* Pin 6 is GND or ground. As described above, this can be connected to pin 5 to reset the microcontroller, or to pin 4 and while connected to pin 4 briefly connect to pin 5 to enter the bootloader mode.

# MIDI generation

The ET-PEB is capable of performing MIDI generation. It does this by capturing writes to the sound chip, and uses the onboard microcontroller to translate these into states of a virtual sound device. The state changes of the virtual sound device are then presented as MIDI messages sent out to serial port of the ET-PEB.

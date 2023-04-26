## STM32F103ZE bootloader using SDIO interface

Functionality:

    - Copies image from the file firmware.bin to FLASH
    - Loading starts at 0x0800 8000 (can be changed)
    - Write protection is automatically removed
    - File/image checksum/crc checking is not done
    - UART1 @115200 can be used to monitor the boot process.
    - FAT32 file system with 512 - 4096 byte AUs.

Load point is set by APP_ADDRESS in bootloader.h.

Image filename is set by CONF_FILENAME in main.h.
Target card was a F103ZE_Pro.

Pinout is at:  https://stm32-base.org/boards/STM32F103ZET6-F103ZE-Board.html

This code is based on the booloader and main routines from:
  https://akospasztor.github.io/stm32-bootloader

Steps to create project:
1) copy already existing project to the new folder
2) delete the Core, FATFS and Middlewares folders
3) create base in STM32cubeIDE & generate code
4) copy Core, FATFS and Middlewares over to new folder
5) update platformio.ini and build.bat
6) update main.h and main.c (compare already existing project vs. new files & copy items as needed)
7) update bootloader files for the new CPU



## Hardware notes:

SDIO is used in low speed polling mode.  SDIO DMA mode worked for cards with
512 byte allocation units but not with 4096 byte allocation units.

This board does NOT have a hardware "SD card is present" pin so some
code was commented out.

Erasing is done via pages. All pages that don't have the boot loader image in it are erased. 

This program comes in at about 35,000 bytes which means it fills up pages 1-17 and extends in to
page 18.  That means the lowest load point is the beginning of page 19 (0x0800 9800). 

APP_ADDRESS can be set to any 512 byte aligned address in any erased page.

Systick was converted to a polling mode.  Just couldn't get the standard interrupt code to work.

Removing write protection on the FLASH pages has two quirks:

    Looks like if one page is to be unprotected then all get unprotected.  The code is written to 
    NOT change the protection of the bootloader but apparently the hardware isn't.
    
    The change in protection actually occurs during hardware reset.  The code does initiate the reset 
    sequence but that just hangs the board.  A power cycle is required to recover functionality.

## Building the image:

There is a shadow project on STM32CubeIDE.  I actually use STM32CubeIDE to compile/build the image.

Platformio within VSCode gives me a "\.sconsign38.dblite: No such file or directory" error message.

Make gives me a "cannot find -lgcc: No such file or directory" error message.  Turns out you're
missing the "libgcc" library.  That library is only available when a 64 bit version of GCC is installed 
that has that unusual option.  Building one is a real pain.

## Porting to another processor:

Porting to other STM32F103 chips is very easy.  The only difference is the
size of the FLASH and SRAM.  You'll need to modify the FLASH defines. 

Porting to other processors requires looking at the page layout, erase
mechanisms and FLASH programming mechanisms. 


BOOT_LOADER_END 08005734 - rev 0.11 FATFS
BOOT_LOADER_END 0800547C - rev 0.13 FATFS
BOOT_LOADER_END 080053FC - rev 0.13 FATFS tiny




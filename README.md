## STM32F103ZE bootloader using SDIO interface

Functionality:

    - Copies image from the file firmware.bin to FLASH
    - Loading starts at 0x0800 8000 (can be changed)
    - Write protection is automatically removed
    - File/image checksum/crc checking is not done
    - UART1 @115200 can be used to monitor the boot process.
    - FAT32 file system with 512 - 4096 byte AUs. exFAT can be enabled

Load point is set by APP_ADDRESS in bootloader.h.

Image filename is set by CONF_FILENAME in main.h.

To enable sxFAT support, go to the file ffconf.h and set _FS_EXFAT to 1 
and set _USE_LFN to 1.  This increases the size of the image enough that the 
lowest point at which an application can be loaded is affected.  See the 
Hardware notes section for details.

Target card was a F103ZE_Pro (has various names such as F103ZE_board) .

Pinout is at:  https://stm32-base.org/boards/STM32F103ZET6-F103ZE-Board.html

This code is based on the booloader and main routines from:
  https://akospasztor.github.io/stm32-bootloader

## Building the image:

This repositry builds in VSCode/platformio as is.

There is a Makefile in the repository. It runs to completion without errors in a msys shell.
The resulting image is about 40% bigger than the VSCode one and it doesn't run.
To get Make to run, two files had to be temporarily added.  In case you want to play
with Make, they are in startup_stm32f103zetx.zip and system_stm32f1xx.zip


## Hardware notes:

SDIO is used in low speed 1 wide polling mode.  Going with higher speeds or 4 bit wide mode resulted in disk I/O errors.

This board does NOT have a hardware "SD card is present" pin so some
code was commented out.

Erasing is done via pages. All pages that don't have the boot loader image in it are erased. 

This program comes in at about 22,300 bytes which means it fills up pages 1-9 and extends in to
page 10.  That means the lowest load point is the beginning of page 11 (0x0800 5800). Enabling 
exFAT support pushes the lowest load point to 0x0800 7800.

APP_ADDRESS can be set to any 512 byte aligned address in any erased page.

Systick was converted to a polling mode.  Just couldn't get the standard interrupt code to work.

Removing write protection on the FLASH pages has two quirks:

    Looks like if one page is to be unprotected then all get unprotected.  The code is written to 
    NOT change the protection of the bootloader but apparently the hardware isn't.
    
    The change in protection actually occurs during hardware reset.  The code does initiate the reset 
    sequence but that just hangs the board.  A power cycle is required to recover functionality.
    
## Software notes:

There are four main files that implement the bootloader functionality:
    main_boot.c
    main_boot.h
    bootloader.c
    bootloader.h

STM32CubeIDE was used to create the non-bootloader code.  After generating code, modify the file 
main.c to invoke main_boot(). Then the folders Core, Drivers, FATFS and Middlewares are copied 
over to the main working directory along with the file STM32F103ZETX_FLASH.ld.  

The minimum object file size is achieved by compiling with VSCode/platformio and setting 
"framework = cmsis" in platformio.ini.  To resolve conflicts, the files "startup_stm32f103zetx.s" 
and "stm32f1xx.h" need to be deleted.  Setting framework to nothing and keeping the files almost
doubles the object size.

STM32CubeIDE can be used to build & debug the program. Copy main_boot.c, main_boot.h, bootloader.c
and bootloader.h into the same directories as main.c and main.h or else STM32CubeIDE won't find them 
during the compile/build process.

If MAKE gives a "cannot find -lgcc: No such file or directory" error message.  This means the GCC system 
being used is missing the "libgcc" library.  That library is only available when a 64 bit version of GCC
is created with the  option.  Building one is a real pain.

The FAT software supplied by STM32CubeIDE, when the STM32F103ZE is selected, does not support exFAT. To get 
exFAT support I copied the FATFS and Middlewares folders from a STM32F407 STM32CubeIDE project and did 
some minor massaging.

## Porting to another processor:

Porting to STM32F103xC and STM32F103xD chips is very easy.  The only difference is the
size of the FLASH and SRAM.  You'll need to modify the FLASH defines. 

Porting to other processors requires looking at the page layout, erase
mechanisms, protect mechanisms and FLASH programming mechanism. 

Steps to create project:
1) copy already existing project to a new folder
2) save main.c and main.h
3) delete the Core, Drivers, FATFS and Middlewares folders
4) create base system in STM32cubeIDE & generate code
5) copy Core, Drivers, FATFS and Middlewares over to the new folder
6) delete the files "startup_stm32f103zetx.s" and "stm32f1xx.h" (they conflict with ones are supplied by the cmsis framework)
7) update platformio.ini and build.bat
8) update main.h and main.c (compare saved files vs. new files & copy items as needed)
9) update bootloader files for the new CPU






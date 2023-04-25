try "C:\install_software\arm-gnu-toolchain-12.2.mpacbti-rel1-mingw-w64-i686-arm-none-eabi\bin\arm-none-eabi-cpp.exe" version  12.2.1 20230214
  no luck
  
  C:/MinGW/bin/cpp.exe --version
        cpp.exe (MinGW.org GCC Build-2) 9.2.0
        
  C:/mingw64/bin/cpp.exe --version
        cpp.exe (x86_64-posix-seh-rev2, Built by MinGW-W64 project) 12.2.0      

If you get the following error then you need to install the 64 bit version of GCC or mingw.  Turns out you're missing the libgcc library but the 32 bit version is only available in the 64 bit package.
    arm-none-eabi/bin/ld.exe: cannot find -lgcc: No such file or directory

It's quite a process to create the 64 bit versions of GCC and mingw on a Windows PC.  Here's the web pages I found most usefull:
    https://wiki.osdev.org/GCC_Cross-Compiler#Note_on_arm-none-eabi-gcc
    https://clang.llvm.org/get_started.html
        The command line that finally worked is: 
           cmake -S llvm -B build -G "Visual Studio 17 2022" -DLLVM_ENABLE_PROJECTS="clang;lld" -DCMAKE_INSTALL_PREFIX=c:/clang_LLVM -DCMAKE_BUILD_TYPE=Releasecd
    https://code.visualstudio.com/docs/cpp/cmake-linux
    https://llvm.org/docs/GettingStarted.html
    https://gnuwin32.sourceforge.net/packages/bison.htm
    https://gnuwin32.sourceforge.net/install.html  (you'll need to use a MSYS shell to run the commands)
    When compiling GMP, initially use the command "configure --host=HOST".  This will give an error but will list 
    the acceptable options.  On my PC I used "configure --host=kabylake-pc-mingw32"kabylake-pc-mingw32.
    
    configure --host=kabylake-pc-mingw32"kabylake-pc-mingw32
      make check gives: FAIL: t-hightomask.exe
    
    
     configure --host=kabylake-pc-mingw32 --enable-cxx --enable-alloca=alloca gmp_cv_asm_x86_mulx=yes
       make check gives: FAIL: t-hightomask.exe
     
     configure --host=kabylake-pc-mingw32"kabylake-pc-mingw32 --enable-cxx --enable-alloca=alloca --disable-static --enable-shared
       goes into a submode - no idea what to do here
       CNTL-C exits it
    
    i686-pc-mingw32 instead of kabylake-pc-mingw32
       same error
       
    switch to gmp_6.1.2
       same error
    
    configure  --build=kabylake-pc-mingw32 --enable-cxx --enable-alloca=alloca gmp_cv_asm_x86_mulx=yes
       unable to find compiler
       
     configure  --build=i686-pc-mingw32 --enable-cxx --enable-alloca=alloca gmp_cv_asm_x86_mulx=yes  
       unable to find compiler
       
     configure --build=x86_64-w64-mingw32 --host=x86_64-w64-mingw32 --disable-static --enable-shared --enable-cxx --enable-alloca=alloca gmp_cv_asm_x86_mulx=yes
       could not find a working compiler
       
       for txinfo
         ./configure --build="x86_64-w64-mingw32" --host="x86_64-w64-mingw32"
         
          AR="C:/mingw64/x86_64-w64-mingw32/bin/ar.exe " LINK="C:/mingw64/bin/ld.exe" ARFLAGS="scr" ./configure --build="x86_64-w64-mingw32" --host="x86_64-w64-mingw32"

          CC="C:/mingw64/bin/gcc.exe" AR="C:/mingw64/bin/ar.exe " LINK="C:/mingw64/bin/ld .exe" ARFLAGS="scr" ./configure --build="x86_64-w64-mingw32" --host="x86_64-w64-mingw32"
          CC="C:/MinGW/bin/gcc.exe" AR="C:/MinGW/bin/ar.exe " LINK="C:/MinGW/bin/ld .exe" ARFLAGS="scr" ./configure --build="x86_64-w64-mingw32" --host="x86_64-w64-mingw32"

          CC="C:/mingw64/bin/gcc.exe" AR="C:/mingw64/bin/ar.exe " LINK="C:/mingw64/bin/ld .exe" ARFLAGS="scr" ./configure --build="x86_64-w64-mingw32" --host="x86_64-w64-mingw32" --enable-cxx

          CC="C:/mingw64/bin/gcc.exe" AR="C:/mingw64/bin/ar.exe " LINK="C:/mingw64/bin/ld .exe" ARFLAGS="scr" ./configure --build="skylake-w64-mingw32" --host="skylake-w64-mingw32" --enable-cxx

          CC="C:/mingw64/bin/gcc.exe" AR="C:/mingw64/bin/ar.exe " LINK="C:/mingw64/bin/ld .exe" ARFLAGS="scr" ./configure --build="skylake-w32-mingw32" --host="skylake-w32-mingw32" --enable-cxx

This one finally got gmp's configure to run to completion
          NM="C:/mingw64/bin/nm.exe" CC="C:/mingw64/bin/gcc.exe" AR="C:/mingw64/bin/ar.exe " LINK="C:/mingw64/bin/ld .exe" ARFLAGS="scr" ./configure --build="skylake-w64-mingw32" --host="skylake-w64-mingw32" --enable-cxx
     
             but make check reports: error adding symbols: archive has no index; run ranlib to add one
          
          NM="C:/mingw64/bin/nm.exe" CC="C:/mingw64/bin/gcc.exe" AR="C:/mingw64/bin/ar.exe scr" LINK="C:/mingw64/bin/ld .exe" ./configure --build="skylake-w64-mingw32" --host="skylake-w64-mingw32" --enable-cxx
          
             now getting issues because final AR_FLAGS is scr cq
          
          changed configure and acinclude.m4 from AR_FLAGS=cq to scr and modified configure command
          NM="C:/mingw64/bin/nm.exe" CC="C:/mingw64/bin/gcc.exe" AR="C:/mingw64/bin/ar.exe" LINK="C:/mingw64/bin/ld .exe" ./configure --build="skylake-w64-mingw32" --host="skylake-w64-mingw32" --enable-cxx
          
          SOB!!! - this configure uses AR_FLAGS but a configure for another program used ARFLAGS
             could have gotten the same results by NOT modifying the two files and using this configure command:
          NM="C:/mingw64/bin/nm.exe" CC="C:/mingw64/bin/gcc.exe" AR="C:/mingw64/bin/ar.exe " LINK="C:/mingw64/bin/ld .exe" AR_FLAGS="scr" ./configure --build="skylake-w64-mingw32" --host="skylake-w64-mingw32" --enable-cxx
        
          make check is using "x86_64-w64-mingw32/bin/ld.exe"
          
          LD="C:/mingw64/bin/ld.exe" AR_FLAGS="scr" make check  
          
          
          LD="C:/mingw64/bin/ld.exe" AR_FLAGS="scr" NM="C:/mingw64/bin/nm.exe" CC="C:/mingw64/bin/gcc.exe" AR="C:/mingw64/bin/ar.exe " LINK="C:/mingw64/bin/ld .exe" ./configure --build="skylake-w64-mingw32" --host="skylake-w64-mingw32" --enable-cxx
        
polylib
          LD="C:/mingw64/bin/ld.exe" AR_FLAGS="scr" NM="C:/mingw64/bin/nm.exe" CC="C:/mingw64/bin/gcc.exe" AR="C:/mingw64/bin/ar.exe " LINK="C:/mingw64/bin/ld .exe" ./configure --build="i686-pc-mingw32" --host="i686-pc-mingw32" --enable-cxx -enable-allint-lib -enable-all-exec
          LD="C:/mingw64/bin/ld.exe" AR_FLAGS="scr" NM="C:/mingw64/bin/nm.exe" CC="C:/mingw64/bin/gcc.exe" AR="C:/mingw64/bin/ar.exe " LINK="C:/mingw64/bin/ld .exe" ./configure --build="i686-pc-cygwin" --host="i686-pc-cygwin" --enable-cxx -enable-allint-lib -enable-all-exec


cd /c/texinfo/info

C:/mingw64/bin/ld.exe -o makedoc.exe makedoc.o ../gnulib/lib/libgnu.a

cd /c/texinfo

## STM32F103ZE bootloader using SDIO interface

Functionality:

    - Copies image from the file firmware.bin to FLASH
    - Loading starts at 0x0800 A000 (can be changed)
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

Removing write protection on the FLASH pages has two quircks:

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
# External loader for STM32G071 and W25Q64
I needed a file system for a small web server managed by an STM32G071 microcontroller. This file system comes in the form of a single file to be copied into an SPI flash managed by the MCU.
How to copy this file in the flash?

## What is an external loader ?
STM has a solution with _STM32CubeProgrammer_, which allows applications to be flashed in the MCU flash, but also to use external loaders.
An external loader (EL) is a small specific application intended to be executed in the RAM of the MCU (therefore without touching its flash application) and which allows operations other than flashing the MCU. The API is very simple:
- Initialization
- Erasing a sector
- Global erase
- Writing of sectors

The EL is written to run on a given microcontroller to access a particular flash on a particular SPI port. It is therefore necessary to develop the EL adapted to each particular application.

## Achievement
My need: Manage a Winbond W25Q64 flash in simple SPI.

There are EL examples provided by STM (https://github.com/STMicroelectronics/stm32-external-loader), but they concern the brand's development boards and therefore QPSPI flashes (quick access with the flash mapped into the memory space of the MCU). Moreover, these examples are reduced to the simplest expression: no way to run them outside of _STM32CubeProgrammer_, and therefore to debug.
See the MOOC: https://www.st.com/content/st_com/en/support/learning/stm32-education/stm32-moocs/external_QSPI_loader.html

After searching everywhere I wrote this external loader.
The application can be generated in two modes:
- In standard mode (with an _.elf_ extension) to be placed in the MCU flash. This allows you to debug comfortably and verify that the 4 functions of the EL are working, as well as the initialization of the MCU and the SPI. Once everything is working, you can switch to EL mode.
- In EL mode (with a _.stldr_ extension), therefore intended to be placed in the MCU's RAM. And in this case you should know that the startup code will not be executed, and therefore the _main()_ either. All initialization must be done in the _Init()_ function.

The change of mode is done by changing the value of _WITH_MAIN_ in _global.h_, and by **changing the script of the linker** (otherwise the behavior can be very strange!). See comments in _global.h_.

A UART is used (without interrupt, that's important) to trace EL API function calls. This UART is connected to the VCP port of the ST-LINK probe, and the traces are displayed in a PuTTY terminal.
Incidentally, an LED indicates the status of the application in standard mode.

The code uses LL rather than HAL, to generate smaller code. It is important to optimize the RAM occupation, which must contain the application and the buffers used by STM32CubeProgrammer to transmit the sectors to be written. And in some MCUs the RAM is small…

This is a TrueSTUDIO project, because I like using it, but the project can be imported into STM32CubeIDE without issue.

Apparently it is important to respect the EL linker script provided by the MOOC. During the development, before finding this MOOC, I had strange behaviors.

Once the EL has been generated, it must be placed where _STM32CubeProgrammer_ can find it:

_C:\Program Files\STMicroelectronics\STM32Cube\STM32CubeProgrammer\bin\ExternalLoader_

For use with STM32CubeProgrammer refer to its documentation.


This EL and [MFL file system](https://adastra-soft.com/a-minimalist-file-system/) are among my basic tools for any small WEB server application on MCU.

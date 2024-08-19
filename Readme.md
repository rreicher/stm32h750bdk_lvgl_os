# STM32H750B-DK with LVGL 8.3.0 and FreeRTOS demo example

![STM32H750B-DK Lvgl 8.3.0!](/img/stm32h750bdk_demo.jpg "")

This repository offers an example of porting the Lvgl demo for the STM32H750B-DK from the existing solution for the STM32H745I-Discovery board.

## How does it work.
The initial project implementation is as follows:
A bootloader (ExtMem_Boot\bootloader.bin) is flashed in internal Flash memory at the address 0x0800 0000. 
This bootloader configures the QSPI external flash in Dual mode which is present on the board.
Finally, it jumps to the external QSPI address 0x9000 0000 to execute the user code and the various Lvgl demos.
The script of the Linker file STM32H750XBHX_FLASH.ld has been modified in order to define the internal Flash area for the bootloader section.
The QSPI defines the user section for the user code and the AXI SRAM stores the data.
The bootloader is loaded automatically when starting the debug sequence. It will just be necessary to reset the board for the bootloader to jump into QSPI memory.
In main function, we create 2 FreeRTOS tasks, lvgl_task() and defaultTask() and one mutex lvglMutexHandle.
The defaultTask toggle a LED.
The lvgl_task() is used to initialize LVGL, LTDC and Touchpad display and execute lvgl demo, and finally continuously
wait/release the lvglMutexHandle. 
TIMER6 is used to manage HAL Tick API, due to Systick timer is reserverd for FreeRTOS scheduler. 

## How to use it.
Clone this repository.
Be sure to exclude the EXMem_Boot directory in STM32CubeIDE and compile th project.
The debugger configuration should include as below:
The external loader MT25TL01G_STM32H750B-DISCO.stldr and it is necessary to position the SP and the PC manually due to the absence of the debugger code.

set *0xE000ED08 = 0x08000000

set $sp = *(unsigned int*)0x08000000

set $pc = *(unsigned int*)0x08000004

![STM32H750B-DK Debugger config external flash loader!](/img/Debug_config.PNG "")

![STM32H750B-DK Debugger config external flash loader!](/img/Debug_startup.PNG "")

## Use STM32CubeMX to configure STM32 peripheral and regenerate the project.
This project is based on the use of STM32H7 HAL LCD BSP drivers. The CubeMX stm32h750bdk_lvgl_nos.ioc file does not configure peripherals like LTDC, DMA2D and I²C4 for the touchpad.
It is possible to modify the configuration of the ioc file to implement other peripherals (ADC, Timer, SPI...)
But in this case it will be necessary to manually copy the following files from STM32Cube_FW_H7_Vx.y.z\Drivers\STM32H7xx_HAL_Driver into their respective directories in the project:
stm32h7xx_hal_dma2d.h
stm32h7xx_hal_dma2d.c
stm32h7xx_ll_dma2d.h
stm32h7xx_hal_ltdc.h
stm32h7xx_hal_ltdc_ex.h
stm32h7xx_hal_ltdc.c
stm32h7xx_hal_ltdc_ex.c

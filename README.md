
# ARM Cortex M Microcontroller DMA Programming Demystified (DMA)

This repo contains notes and programming assignments for the Udemy's "[ARM Cortex M Microcontroller DMA Programming Demystified (DMA)](https://www.udemy.com/course/microcontroller-dma-programming-fundamentals-to-advanced/?couponCode=KEEPLEARNING)" course by FastBit Embedded Brain Academy.

Date: March, 2024.

- The course is instructed by Engineer Kiran Nayak.

- The [**Certificate**](https://github.com/renatosoriano/Udemy-Embedded-Course8_ARM-Cortex-M-Microcontroller-DMA-Programming-Demystified/blob/main/Certificate.pdf) is available. 

## Descriptions

The course aims to demystify the Microcontroller DMA controller internals and its programming with various peripherals. It is discussed why DMA is required and how it benefits ARM to offload data transfer work with exercises. We learn different types of DMA transfers like M2M, P2M, and M2P (M: Memory P: Peripheral) and various DMA configurations.

The course covers:
1. The Multi AHB bus matrix and ARM Cortex M Bus interfaces.
2. MCU Master and Slave communication over bus matrix.
3. DMA internals: channel mapping / streams / fifo / Master ports / Arbiter / etc.
4. DMA different transfer modes: M2P, P2M, M2M.
5. DMA with peripherals like ADC, GPIO, UART_RX/TX and many other peripherals.
6. DMA programming from scratch.

## Requirements

**[STM32 Nucleo-F446RE Development Board](https://www.st.com/en/evaluation-tools/nucleo-f446re.html#overview)** - Board used in this course but any board with Arm Cortex-M0/3/4 core will work, just modifying the target board and configuring with the respective datasheet. \
**[Eclipse-based STM32CubeIDE](https://www.st.com/en/development-tools/stm32cubeide.html)** - C/C++ development platform with peripheral configuration, code generation, code compilation, and debug features for STM32 microcontrollers and microprocessors. Works on Windows/Linux/Mac and is free. \

## Notes
* #### FPU warning fix
    Right click on the project -> properties -> expand C/C++ build -> Settings -> Tool settings -> MCU settings
  * `Floating-point unit: None`
  * `Floating-point ABI: Software implementation ( -mfloat-abi=soft )`

![FPU_warning.png](https://github.com/renatosoriano/Udemy-Embedded-Course8_ARM-Cortex-M-Microcontroller-DMA-Programming-Demystified/blob/main/Images/FPU_warning.png)

* #### Setting up SWV ITM Data Console

Open *syscalls.c* file and paste following code bellow *Includes*

```c
/////////////////////////////////////////////////////////////////////////////////////////////////////////
//           Implementation of printf like feature using ARM Cortex M3/M4/ ITM functionality
//           This function will not work for ARM Cortex M0/M0+
//           If you are using Cortex M0, then you can use semihosting feature of openOCD
/////////////////////////////////////////////////////////////////////////////////////////////////////////


//Debug Exception and Monitor Control Register base address
#define DEMCR                   *((volatile uint32_t*) 0xE000EDFCU )

/* ITM register addresses */
#define ITM_STIMULUS_PORT0   	*((volatile uint32_t*) 0xE0000000 )
#define ITM_TRACE_EN          	*((volatile uint32_t*) 0xE0000E00 )

void ITM_SendChar(uint8_t ch)
{

	//Enable TRCENA
	DEMCR |= ( 1 << 24);

	//enable stimulus port 0
	ITM_TRACE_EN |= ( 1 << 0);

	// read FIFO status in bit [0]:
	while(!(ITM_STIMULUS_PORT0 & 1));

	//Write to ITM stimulus port0
	ITM_STIMULUS_PORT0 = ch;
}
```


After that find function *_write* and replace `__io_putchar(*ptr++)` with `ITM_SendChar(*ptr++)` like in code snippet below
```c
__attribute__((weak)) int _write(int file, char *ptr, int len)
{
	int DataIdx;

	for (DataIdx = 0; DataIdx < len; DataIdx++)
	{
		//__io_putchar(*ptr++);
		ITM_SendChar(*ptr++);
	}
	return len;
}
```

After these steps navigate to Debug configuration and check `Serial Wire Viewer (SWV)` check box like on snapshot below

![Debugger.png](https://github.com/renatosoriano/Udemy-Embedded-Course8_ARM-Cortex-M-Microcontroller-DMA-Programming-Demystified/blob/main/Images/Debugger.png)

Once you enter *Debug* mode, go to `Window -> Show View -> SWV -> Select SWV ITM Data Console`. On this way `ITM Data Console` will be shown in *Debug* session.


In `SWV ITM Data Console Settings` in section `ITM Stimulus Ports` enable port 0, so that you can see `printf` data.




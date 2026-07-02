# Embedded Exponential Sine Sweep Analyzer

## Overview

This project implements a **portable STM32-based embedded system** for **acoustic frequency response measurements** using the **Exponential Sine Sweep (ESS) method**.

The system is designed to characterize audio systems through frequency sweep analysis, capturing and processing acoustic signals on an embedded microcontroller platform.

## Key Features

- **STM32L476RG Microcontroller** (NUCLEO-L476RG Board)
- **Exponential Sine Sweep (ESS) analysis** for acoustic measurements
- **SAI2 interface** for high-quality audio I/O (48 kHz sampling)
- **DMA-enabled data transfer** for efficient processing
- **I2C communication** for sensor integration
- **UART interface** at 921600 baud for data transmission
- **DSP capabilities** using ARM CMSIS-DSP library
- **Real-time signal processing** with interrupt-driven architecture

## Hardware Requirements

### Microcontroller
- **MCU**: STM32L476RGTx
- **Board**: NUCLEO-L476RG
- **CPU Clock**: 80 MHz
- **Flash Memory**: 1024 KB
- **RAM**: 96 KB + 32 KB (RAM2)
- **Floating Point Unit**: FPv4-SP-D16

### Peripherals
- **SAI2 (Serial Audio Interface)**
  - Master mode (transmission)
  - Synchronized slave mode (reception)
  - Audio frequency: 48 kHz
  - DMA circular mode for continuous streaming
- **I2C1**: Standard mode (100 kHz)
- **USART2**: 921600 baud (for debug/data output)
- **DMA1, DMA2**: For SAI and UART transfers

## Pin Configuration

| Pin | Signal | Function | Mode |
|-----|--------|----------|------|
| PA2 | USART_TX | Serial transmission | AF7 |
| PA3 | USART_RX | Serial reception | AF7 |
| PA5 | LD2 | Green LED | Output |
| PB8 | I2C1_SCL | I2C clock | AF4 |
| PB9 | I2C1_SDA | I2C data | AF4 |
| PB12 | SAI2_FS_A | Audio frame sync | AF13 |
| PB13 | SAI2_SCK_A | Audio clock | AF13 |
| PB15 | SAI2_SD_A | Audio data out | AF13 |
| PC9 | SAI2_EXTCLK | External audio clock | AF3 |
| PC12 | SAI2_SD_B | Audio data in | AF13 |
| PC13 | B1 | Blue push button | GPIO_IT |

## Project Structure

```
├── Core/
│   ├── Inc/                    # Header files
│   │   ├── main.h
│   │   ├── gpio.h
│   │   ├── dma.h
│   │   ├── i2c.h
│   │   ├── sai.h
│   │   ├── usart.h
│   │   ├── stm32l4xx_it.h     # Interrupt handlers
│   │   └── stm32l4xx_hal_conf.h
│   └── Src/                    # Source files
│       ├── main.c
│       ├── gpio.c
│       ├── dma.c
│       ├── i2c.c
│       ├── sai.c
│       ├── usart.c
│       ├── stm32l4xx_it.c
│       ├── stm32l4xx_hal_msp.c
│       └── system_stm32l4xx.c
├── Drivers/                    # STM32 HAL and CMSIS
│   ├── STM32L4xx_HAL_Driver/
│   └── CMSIS/
├── Build/                      # Build artifacts
│   ├── Debug/
│   └── Release/
├── Documentation/              # Project documentation
│   ├── HARDWARE.md
│   ├── SETUP.md
│   └── ESS_METHOD.md
├── .cproject                   # Eclipse CDT project file
├── .project                    # Eclipse project file
├── DSP_project_ac.ioc         # STM32CubeMX configuration
├── DSP_project_ac.launch      # Debug launch configuration
├── STM32L476RGTX_FLASH.ld     # Flash linker script
├── STM32L476RGTX_RAM.ld       # RAM linker script
├── Makefile                    # Build configuration
└── README.md                   # This file
```

## Software Stack

- **HAL Layer**: STM32L4xx HAL Driver Library
- **CMSIS**: ARM Cortex-M4 CMSIS Core
- **DSP**: ARM CMSIS-DSP Library (for FFT and signal processing)
- **Compiler**: ARM GCC (arm-none-eabi-gcc)
- **IDE**: STM32CubeIDE

## Build Instructions

### Using STM32CubeIDE
1. Import project into STM32CubeIDE
2. Build: Project → Build Project (Ctrl+B)
3. Debug: Run → Debug (F11)

### Using Command Line
```bash
make -j$(nproc)              # Build with optimization
make DEBUG=1                 # Build with debug symbols
make clean                   # Clean build artifacts
```

### Output
- Debug build: `Debug/DSP_project_ac.elf`
- Release build: `Release/DSP_project_ac.elf`
- Hex file: `DSP_project_ac.hex`
- Binary: `DSP_project_ac.bin`

## Programming the MCU

### Using ST-Link
```bash
st-flash write build/Release/DSP_project_ac.bin 0x8000000
```

### Using STM32CubeIDE
1. Connect ST-Link debugger
2. Run → Debug As → Embedded C/C++ Application

## Serial Communication

- **Port**: /dev/ttyUSB0 (Linux) or COM3 (Windows)
- **Baud Rate**: 921600
- **Data Bits**: 8
- **Stop Bits**: 1
- **Parity**: None
- **Flow Control**: None

## Features Implementation

### Exponential Sine Sweep (ESS)
The ESS method sweeps through a frequency range using an exponential chirp signal. This provides:
- Better frequency resolution at lower frequencies
- Optimized SNR across the entire frequency range
- Reduced measurement time

### Real-Time Processing
- SAI2 DMA transfers audio data continuously
- Circular DMA buffers prevent data loss
- Interrupt-driven processing pipeline
- Efficient memory utilization

### Clock Configuration
- **System Clock**: 80 MHz (from PLL)
- **Audio Clock**: 12.288 MHz external clock
- **I2C Clock**: 100 kHz
- **UART Clock**: 921600 baud

## Debugging

Enable serial debug output via UART2:
```c
printf("System initialized\n");
printf("SAI audio started at %d Hz\n", AUDIO_FREQ);
```

## Performance Characteristics

| Parameter | Value |
|-----------|-------|
| Sampling Rate | 48 kHz |
| Audio Format | PCM 16-bit |
| DMA Buffer Size | TBD (configurable) |
| Frequency Range | 20 Hz - 20 kHz (ESS configurable) |
| Resolution | ±0.1 dB (nominal) |

## License

This project is licensed under the MIT License - see LICENSE file for details.

## Documentation

For more detailed information, see:
- [Hardware Setup](Documentation/HARDWARE.md)
- [Development Setup](Documentation/SETUP.md)
- [ESS Method Theory](Documentation/ESS_METHOD.md)

## Support

For issues, feature requests, or contributions, please use the GitHub Issues and Pull Requests sections.

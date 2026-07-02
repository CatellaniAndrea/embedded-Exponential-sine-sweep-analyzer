# Hardware Setup Guide

## Overview

This document details the hardware configuration and pin layout for the embedded ESS analyzer.

## NUCLEO-L476RG Board

The project uses the **STM32 NUCLEO-L476RG** development board featuring:
- STM32L476RGTx microcontroller
- 64-pin LQFP package
- Built-in ST-Link debugger
- Power regulation and reset circuit

## MCU Specifications

| Feature | Value |
|---------|-------|
| Architecture | ARM Cortex-M4 with FPU |
| Max Clock | 80 MHz |
| Flash Memory | 1024 KB |
| SRAM | 96 KB (RAM) + 32 KB (RAM2) |
| Floating Point Unit | FPv4-SP-D16 (single precision) |
| Power Supply | 3.3V |

## Pin Configuration

### UART Interface (USART2)
- **TX**: PA2 (AF7) - Data transmission
- **RX**: PA3 (AF7) - Data reception
- **Baud Rate**: 921600
- **Used for**: Debug output, data logging, serial communication

### I2C Interface (I2C1)
- **SCL**: PB8 (AF4, Pull-up enabled)
- **SDA**: PB9 (AF4, Pull-up enabled)
- **Speed**: Standard mode (100 kHz)
- **Used for**: Sensor communication (e.g., audio codec configuration)

### SAI2 Audio Interface
The SAI2 (Serial Audio Interface) is configured in master/slave synchronous mode:

#### SAI2 Block A (Master)
- **FS_A** (Frame Sync): PB12 (AF13) - 48 kHz frame clock
- **SCK_A** (Serial Clock): PB13 (AF13) - Audio bit clock
- **SD_A** (Serial Data): PB15 (AF13) - Audio output data
- **Mode**: Master, PCM format, 16-bit samples
- **Frequency**: 48 kHz
- **DMA**: DMA1 Channel 6 (Memory to Peripheral, circular mode)

#### SAI2 Block B (Slave)
- **SD_B** (Serial Data): PC12 (AF13) - Audio input data
- **Mode**: Slave, synchronized to Block A
- **DMA**: DMA2 Channel 4 (Peripheral to Memory, circular mode)

#### External Audio Clock
- **EXTCLK**: PC9 (AF3)
- **Frequency**: 12.288 MHz (external oscillator)

### GPIO Outputs
- **LD2** (Green LED): PA5 - Status indicator
- **B1** (Blue Button): PC13 - User input/trigger

## Clock Configuration

### System Clock
- **Source**: PLL from internal HSI (16 MHz)
- **Multiplier**: 10
- **Divider**: 2
- **Result**: 80 MHz system clock

### Peripheral Clocks
| Peripheral | Frequency | Notes |
|------------|-----------|-------|
| APB1/APB2 | 80 MHz | UART, I2C, SAI, DMA |
| I2C1 | 80 MHz | Standard mode timing |
| SAI2 | 12.288 MHz | External clock input |
| USART2 | 80 MHz | Baud rate generator |

### Clock Sources
- **HSI**: 16 MHz internal RC oscillator
- **HSE**: 12.288 MHz external crystal (for audio)
- **LSE**: 32 kHz low-speed oscillator (optional, for RTC)

## DMA Configuration

### DMA1 Channel 6 - SAI2_A TX
- **Source**: Memory (audio output buffer)
- **Destination**: SAI2 data register
- **Mode**: Circular (continuous)
- **Data Width**: 16-bit
- **Priority**: Very High
- **Trigger**: SAI2_A transmission request

### DMA2 Channel 4 - SAI2_B RX
- **Source**: SAI2 data register
- **Destination**: Memory (audio input buffer)
- **Mode**: Circular (continuous)
- **Data Width**: 16-bit
- **Priority**: Very High
- **Trigger**: SAI2_B reception request

### DMA1 Channel 7 - USART2 TX
- **Source**: Memory (TX buffer)
- **Destination**: USART2 data register
- **Mode**: Normal (one-shot)
- **Data Width**: 8-bit
- **Priority**: Low
- **Trigger**: USART2 transmission request

## Interrupt Configuration

| Interrupt | Priority | Handler | Purpose |
|-----------|----------|---------|----------|
| SAI2_IRQn | - | (DMA driven) | Audio interface errors |
| DMA1_Channel6_IRQn | 2 | DMA_IRQHandler | SAI TX half/full transfer |
| DMA2_Channel4_IRQn | 2 | DMA_IRQHandler | SAI RX half/full transfer |
| DMA1_Channel7_IRQn | 2 | DMA_IRQHandler | UART TX complete |
| USART2_IRQn | 5 | USART2_IRQHandler | UART RX data |
| EXTI15_10_IRQn | 7 | EXTI_IRQHandler | Button (PC13) interrupt |

## External Components

### Audio Codec (Optional)
For full audio I/O, connect an external audio codec:
- **Interface**: I2C1 for configuration
- **Audio Lines**: SAI2 (differential or single-ended)
- **Power**: 3.3V
- **Examples**: WM8731, ES8388, ADAU1701

### Oscillator Module (12.288 MHz)
Required for precise audio timing:
- **Frequency**: 12.288 MHz ±50 ppm
- **Type**: Crystal or oscillator module
- **Load Capacitance**: As per datasheet
- **Output**: PC9 (SAI2_EXTCLK)

### Decoupling Capacitors
- **Main Power (3.3V)**: 100 nF × 4 (per datasheet)
- **Analog Power**: 10 µF + 100 nF
- **Oscillator**: 10-20 pF load capacitors

## Power Supply

### Voltage Levels
- **IO Supply**: 3.3V
- **Analog Supply**: 3.3V
- **Core Supply**: Generated internally

### Power Modes
- **Run Mode**: ~20 mA at 80 MHz
- **Sleep Mode**: ~2 mA (configurable)
- **Stop Mode**: <1 µA (with RTC)

## Debug Interface

### ST-Link
- **Interface**: SWD (Serial Wire Debug)
- **Connector**: Built-in on NUCLEO board
- **Speed**: Configurable (default 1.8 MHz)
- **Functions**: Debugging, programming, serial port emulation

## External Connections

### USB
- **Type**: Micro-USB
- **Purpose**: Power, ST-Link communication, virtual COM port
- **Power**: 500 mA available

### Audio Connectors
For actual audio measurements:
- **Input**: Analog or digital (SAI2_SD_B)
- **Output**: Analog or digital (SAI2_SD_A)
- **Impedance Matching**: Per application requirements

## Measurement Considerations

### Signal Chain
1. Audio Input → SAI2_SD_B → DMA2 → Memory
2. Memory → Processing → UART2_TX → DMA1 → Serial
3. Memory → DMA1 → SAI2_SD_A → Audio Output

### Dynamic Range
- **Input Resolution**: 16-bit ADC (external codec)
- **Processing**: 32-bit fixed or floating point
- **Output Resolution**: 16-bit DAC (external codec)

### Frequency Response
- **SAI Sample Rate**: 48 kHz
- **Nyquist Frequency**: 24 kHz
- **Practical Measurement Range**: 20 Hz - 20 kHz (ESS configurable)

## Schematic Recommendations

See project repository for detailed schematic including:
- Power distribution
- Clock networks
- Decoupling strategy
- Audio interface connections
- Optional peripheral connections

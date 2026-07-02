# Development Environment Setup

## Prerequisites

### Required Tools
1. **STM32CubeIDE** (recommended) or compatible IDE
2. **ARM GCC Compiler** (arm-none-eabi-gcc)
3. **ST-Link Debugger** (included with NUCLEO board)
4. **USB Cable** (Micro-USB for board programming)

### Supported Operating Systems
- Windows 10/11
- Linux (Ubuntu 18.04+)
- macOS 10.14+

## Installation Guide

### Windows

#### 1. STM32CubeIDE
```bash
# Download from https://www.st.com/en/development-tools/stm32cubeide.html
# Run installer and follow on-screen instructions
# Default installation path: C:\Program Files\STMicroelectronics\STM32CubeIDE
```

#### 2. ST-Link Drivers
```bash
# Download STM32 ST-Link Utility from ST website
# Install USB driver from STM32CubeMX drivers folder
# Verify: Device Manager should show STM32 STLink
```

#### 3. Serial Port Driver
```bash
# STM32CubeIDE includes virtual COM port driver
# After connection, check Device Manager for COM port number
# Typical: COM3, COM4, etc.
```

### Linux (Ubuntu)

#### 1. Install ARM GCC
```bash
sudo apt-get update
sudo apt-get install gcc-arm-none-eabi
sudo apt-get install libnewlib-arm-none-eabi
sudo apt-get install binutils-arm-none-eabi
```

#### 2. Install STM32CubeIDE
```bash
# Download from ST website
tar -xzf STM32CubeIDE-Linux-*.tar.gz
cd STM32CubeIDE
./STM32CubeIDE
```

#### 3. ST-Link Drivers
```bash
sudo apt-get install libusb-1.0-0-dev
git clone https://github.com/texane/stlink.git
cd stlink
make
sudo make install
sudo ldconfig
```

#### 4. USB Permissions
```bash
# Allow non-root access to ST-Link
sudo usermod -a -G dialout $USER
sudo usermod -a -G plugdev $USER
# Add udev rules
sudo cp <stlink>/config/udev/rules.d/49-stlinkv*.rules /etc/udev/rules.d/
sudo udevadm control --reload-rules
```

### macOS

#### 1. Install Homebrew
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

#### 2. Install ARM GCC
```bash
brew install arm-none-eabi-gcc
```

#### 3. STM32CubeIDE
```bash
# Download DMG from ST website
# Standard installation to /Applications
```

## Project Import

### Using STM32CubeIDE

1. **Launch STM32CubeIDE**
   - Select workspace directory
   - Accept license if first run

2. **Import Project**
   - File → Import → General → Existing Projects into Workspace
   - Select project root directory
   - Finish

3. **Verify Configuration**
   - Right-click project → Properties
   - Check: Target → Device = STM32L476RGTx
   - Check: C/C++ Build → Toolchain = MCU ARM GCC

### Using Command Line

```bash
# Clone repository
git clone https://github.com/CatellaniAndrea/embedded-Exponential-sine-sweep-analyzer.git
cd embedded-Exponential-sine-sweep-analyzer

# Build
make -j4

# Generate hex file
arm-none-eabi-objcopy -O ihex Debug/DSP_project_ac.elf DSP_project_ac.hex
```

## Building the Project

### STM32CubeIDE Build

1. **Clean Build**
   - Project → Clean → Select project → OK

2. **Build**
   - Project → Build Project (or Ctrl+B)
   - Check console for build messages
   - Should complete without errors

3. **Output Files**
   - Debug/DSP_project_ac.elf - Executable with debug symbols
   - Debug/DSP_project_ac.bin - Binary image
   - Debug/DSP_project_ac.hex - Hex format

### Build Configuration

#### Debug Build
```
Optimization: -O0 (no optimization)
Debug Symbols: -g3 (maximum)
File Size: ~500 KB
```

#### Release Build
```
Optimization: -Os (size)
Debug Symbols: None
File Size: ~200 KB
```

## Programming the MCU

### Using STM32CubeIDE

1. **Connect Board**
   - Plug in ST-Link USB cable
   - Wait for driver installation (~10 seconds)

2. **Program**
   - Run → Debug As → Embedded C/C++ Application
   - Or Run → Run As → Embedded C/C++ Application
   - Code automatically flashes and starts

3. **Verify**
   - Green LED should blink
   - Serial port should show startup messages

### Command Line

```bash
# Using st-flash
st-flash write Debug/DSP_project_ac.bin 0x8000000

# Verify flash
st-flash dump 0x8000000 0x400 verified.bin
```

## Debugging

### Debug Session

1. **Start Debug**
   - Run → Debug Configurations
   - Select "embedded-Exponential-sine-sweep-analyzer Debug"
   - Debug

2. **Debug Controls**
   - F5: Resume execution
   - F6: Step over
   - F5: Step into
   - Ctrl+Shift+B: Toggle breakpoint

3. **Breakpoints**
   - Double-click line number to set
   - View → Breakpoints for management
   - Can condition breakpoints (hit count, expression)

4. **Watch Variables**
   - View → Expressions
   - Add variables to watch
   - Real-time value updates

### Serial Debugging

1. **Connect Serial Terminal**
   - Windows: PuTTY, Tera Term (921600 baud)
   - Linux: minicom, picocom
   - macOS: screen

2. **minicom Setup (Linux)**
   ```bash
   sudo minicom -s
   # Configure: /dev/ttyUSB0, 921600 8N1
   # Save as 'dfl', exit
   minicom
   ```

3. **Expected Output**
   ```
   System initialized
   SAI audio interface started
   DMA transfers: OK
   ESS analyzer ready
   ```

## Troubleshooting

### Build Issues

**Error: "Cannot find -lgcc"**
- Solution: Verify arm-none-eabi-gcc installation
- Check: which arm-none-eabi-gcc

**Error: "undefined reference to 'main'"**
- Solution: Ensure Core/Src/main.c is included
- Check project properties for source paths

### Programming Issues

**ST-Link not recognized**
- Windows: Reinstall ST-Link drivers
- Linux: Check udev rules and USB permissions
- macOS: Verify STM32CubeIDE has USB access

**Timeout during programming**
- Check USB cable quality
- Try different USB port
- Reset board manually before programming

### Debug Issues

**"GDB connection timeout"**
- Power cycle board
- Check ST-Link firmware (update if needed)
- Try OpenOCD instead of STM32CubeProgrammer

**Breakpoints not stopping**
- Flash may be in wrong state
- Perform full erase via STM32CubeProgrammer
- Verify debug build selected

## Performance Optimization

### Compiler Flags

| Flag | Effect | Use Case |
|------|--------|----------|
| -Os | Optimize for size | Limited flash (default) |
| -O2 | Optimize for speed | DSP processing |
| -O3 | Aggressive optimization | Performance critical |
| -LTO | Link-time optimization | Combined with others |

### Memory Optimization

- Use CCRAM (core-coupled RAM) for critical loops
- Place constants in FLASH with --specs=nosys.specs
- Enable compiler optimizations for small code

## Next Steps

1. Review [HARDWARE.md](HARDWARE.md) for pin configuration
2. Read [ESS_METHOD.md](ESS_METHOD.md) for algorithm overview
3. Start with [Core/Src/main.c](../Core/Src/main.c) for code structure
4. Run example: Compile → Program → Debug → Observe serial output

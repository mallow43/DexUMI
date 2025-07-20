# DexUMI STM32F103 Firmware

This firmware is a port of the original DexUMI project from STM32F0 to STM32F103, designed for high-precision sensor data acquisition using dual ADS1256 24-bit ADCs.

## 🎯 Overview

The DexUMI STM32F103 firmware collects sensor data from up to 14 channels (XHAND configuration) using two ADS1256 ADC chips, processes the data with checksum validation and timestamping, then transmits it via UART at 921.6 kbps.

## 🔧 Hardware Requirements

### STM32F103 Microcontroller
- **Target MCU**: STM32F103CBT6 (or compatible STM32F103 variant)
- **Package**: LQFP48
- **Flash**: 128KB minimum
- **RAM**: 20KB minimum

### External Components
- **2x ADS1256** - 24-bit Delta-Sigma ADC chips
- **External Crystal** - 8MHz HSE oscillator
- **LEDs** - Status indicators (LED1, LED2)
- **UART Interface** - For data transmission (921.6 kbps)

## 📋 Pin Configuration

| Pin | Function | Description |
|-----|----------|-------------|
| **PA5** | SPI1_SCK | SPI Clock for ADS1256 |
| **PA6** | SPI1_MISO | SPI Data In from ADS1256 |
| **PA7** | SPI1_MOSI | SPI Data Out to ADS1256 |
| **PA9** | USART1_TX | UART Transmit |
| **PA10** | USART1_RX | UART Receive |
| **PA11** | SPI1_CS1 | Chip Select for ADS1256 #1 |
| **PA12** | SPI1_CS2 | Chip Select for ADS1256 #2 |
| **PA15** | LED | General Status LED |
| **PB0** | ADS1256_DRDY_1 | Data Ready from ADS1256 #1 |
| **PB1** | ADS1256_DRDY_2 | Data Ready from ADS1256 #2 |
| **PB4** | LED1 | Status LED 1 |
| **PB5** | LED2 | Status LED 2 |
| **PA0-PA4** | ADC1_IN0-4 | Optional ADC inputs (currently unused) |

## 🚀 Quick Start

### 1. Build the Firmware

```bash
# Navigate to the firmware directory
cd DexUMI_F103

# Clean and build
make clean && make

# Verify build success - you should see:
# arm-none-eabi-size build/DexUMI_F103.elf
#    text    data     bss     dec     hex filename
#    6672      12    1860    8544    2160 build/DexUMI_F103.elf
```

### 2. Flash the Firmware

Use your preferred STM32 programming tool:

**Using ST-Link:**
```bash
st-flash write build/DexUMI_F103.bin 0x8000000
```

**Using OpenOCD:**
```bash
openocd -f interface/stlink.cfg -f target/stm32f1x.cfg -c "program build/DexUMI_F103.elf verify reset exit"
```

**Using STM32CubeProgrammer:**
- Load `build/DexUMI_F103.hex`
- Program to address `0x8000000`

### 3. Hardware Connections

1. **Power Supply**: Connect 3.3V and GND
2. **Crystal**: Connect 8MHz crystal to PD0/PD1 (HSE)
3. **ADS1256 Chips**: 
   - Connect SPI lines (SCK, MISO, MOSI)
   - Connect CS1 to first ADS1256, CS2 to second
   - Connect DRDY pins to PB0 and PB1
4. **UART**: Connect PA9 (TX) to your data receiver
5. **LEDs**: Connect status LEDs to PB4 and PB5

## 📊 Data Format

The firmware transmits sensor data packets via UART:

### XHAND Configuration (64 bytes)
```
Byte 0-3:   Header (0x55AA0000)
Byte 4-7:   Channel 1 data (24-bit ADC + padding)
Byte 8-11:  Channel 2 data
...
Byte 56-59: Channel 14 data
Byte 60-63: Checksum (16-bit) + Timestamp (16-bit)
```

### Standard Configuration (40 bytes)
```
Byte 0-3:   Header (0x55AA0000)
Byte 4-7:   Channel 1 data
...
Byte 32-35: Channel 8 data
Byte 36-39: Checksum (16-bit) + Timestamp (16-bit)
```

## ⚙️ Configuration Options

### Build Configurations

**XHAND Mode (Default):**
```c
#define BUILD_FOR_XHAND  // Enables 14-channel mode
```

**Standard Mode:**
```c
// Comment out BUILD_FOR_XHAND for 8-channel mode
```

### UART Settings
- **Baud Rate**: 921,600 bps
- **Data Bits**: 8
- **Stop Bits**: 1
- **Parity**: None
- **Flow Control**: None

### SPI Settings (ADS1256)
- **Mode**: Master
- **Clock**: 2.25 MHz
- **Phase**: 2nd Edge
- **Polarity**: Low

## 🔍 Troubleshooting

### Build Issues

**Error: `arm-none-eabi-gcc: command not found`**
```bash
# Install ARM GCC toolchain
brew install --cask gcc-arm-embedded  # macOS
# or
sudo apt-get install gcc-arm-none-eabi  # Ubuntu/Debian
```

**Error: Missing header files**
- Ensure all files in `Users/` directory are present:
  - `ads1256.h` and `ads1256.c`
  - `delay.h` and `delay.c`
  - `fsr.h` and `fsr.c`

### Runtime Issues

**No UART Output:**
1. Check UART connections (PA9 = TX)
2. Verify baud rate (921,600 bps)
3. Ensure crystal oscillator is working (8MHz HSE)

**LEDs Not Working:**
1. Check LED connections to PB4 (LED1) and PB5 (LED2)
2. LEDs should turn on during initialization

**ADS1256 Communication Issues:**
1. Verify SPI connections (PA5, PA6, PA7)
2. Check chip select pins (PA11, PA12)
3. Ensure DRDY pins are connected (PB0, PB1)
4. Verify ADS1256 power supply (5V or 3.3V depending on variant)

## 📁 Project Structure

```
DexUMI_F103/
├── Core/
│   ├── Inc/           # Header files
│   └── Src/           # Source files
├── Drivers/           # STM32 HAL drivers
├── Users/             # Custom drivers
│   ├── ads1256.c/h    # ADS1256 ADC driver
│   ├── delay.c/h      # Timing utilities
│   └── fsr.c/h        # Force sensor routines
├── build/             # Build output
├── Makefile           # Build configuration
├── DexUMI_F103.ioc    # STM32CubeMX project
└── README.md          # This file
```

## 🔄 Migration from STM32F0

This firmware is functionally equivalent to the original STM32F0 version with the following improvements:

- **Higher Performance**: STM32F103 Cortex-M3 @ 72MHz vs STM32F0 Cortex-M0 @ 48MHz
- **More Memory**: 128KB Flash / 20KB RAM vs 32KB Flash / 6KB RAM
- **Enhanced Peripherals**: More advanced timers and additional peripherals
- **Same Application Logic**: Identical sensor data collection and processing

## 📞 Support

For issues related to:
- **Hardware**: Check pin connections and power supply
- **Build**: Verify toolchain installation and file structure
- **Communication**: Test UART settings and connections
- **ADS1256**: Refer to ADS1256 datasheet for chip-specific issues

## 📄 License

This firmware is part of the DexUMI project. Refer to the main project LICENSE file for licensing information.

---

**Version**: 1.0  
**Target**: STM32F103CBT6  
**Build Date**: Generated from STM32CubeMX  
**Compatibility**: Functionally equivalent to STM32F0 version

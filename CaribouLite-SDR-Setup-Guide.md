# CaribouLite SDR Setup Guide for Raspberry Pi

This guide documents the complete setup process for getting CaribouLite working with SDR software on Raspberry Pi.

## Overview

This guide covers:
- Fixing gqrx crashes with CaribouLite
- Installing and building SDR++ Brown from source with SoapySDR support
- Troubleshooting plugin compatibility issues

## Hardware

- **Device**: CaribouLite S1G (Sub-1GHz) Rev2.8
- **Serial**: 2461461835
- **Frequency Range**: 389.5-510 MHz and 779-1020 MHz
- **Sample Rates**: 0.4 to 4 MSPS
- **RX Gain**: 0-69 dB
- **TX Gain**: 0-31 dB

## Issue 1: gqrx Crashes on Startup

### Problem
```bash
$ gqrx
# Results in SIGSEGV (Segmentation Fault)
CaribouLite: Signal [11] received from pid=[16]
SIGSEGV: memory access violation
```

### Root Cause
The SoapyCariboulite driver has a bug that causes gqrx to crash during device enumeration at startup.

### Solution
Temporarily disable the SoapyCariboulite module for gqrx:

```bash
sudo mv /usr/lib/aarch64-linux-gnu/SoapySDR/modules0.8/libSoapyCariboulite.so \
         /usr/lib/aarch64-linux-gnu/SoapySDR/modules0.8/libSoapyCariboulite.so.disabled
```

To re-enable later:
```bash
sudo mv /usr/lib/aarch64-linux-gnu/SoapySDR/modules0.8/libSoapyCariboulite.so.disabled \
         /usr/lib/aarch64-linux-gnu/SoapySDR/modules0.8/libSoapyCariboulite.so
```

### Verification
Verify CaribouLite driver works with SoapySDR directly:

```bash
SoapySDRUtil --probe=driver=Cariboulite
```

Expected output:
```
driver=Cariboulite
hardware=Cariboulite Rev2.8
product_name=CaribouLite RPI Hat
Channels: 1 Rx, 1 Tx
```

## Issue 2: Installing SDR++ Brown with SoapySDR Support

### Why SDR++ Brown?
SDR++ Brown is a feature-rich fork that includes:
- Built-in FT8/FT4 decoder
- Advanced noise reduction (OMLSA MCRA algorithm)
- Hermes Lite 2 support with TX capability
- KiwiSDR integration
- Enhanced UI performance
- Better mobile/4K display support

### Prerequisites

Install build dependencies:

```bash
sudo apt install -y cmake libad9361-dev libairspy-dev libairspyhf-dev \
    libfftw3-dev libglfw3-dev libhackrf-dev libiio-dev librtaudio-dev \
    libvolk-dev libzstd-dev git build-essential libsoapysdr-dev soapysdr-tools
```

### Build Process

1. **Clone the repository:**

```bash
cd ~
git clone https://github.com/sannysanoff/SDRPlusPlusBrown.git
```

2. **Configure with SoapySDR support:**

```bash
cd SDRPlusPlusBrown
mkdir build
cd build
cmake -DOPT_BUILD_SOAPY_SOURCE=ON ..
```

**Note**: CMake will take several minutes as it builds bundled dependencies (itpp, mbelib).

3. **Build (single-threaded to avoid overwhelming Raspberry Pi):**

```bash
make -j1
```

**Note**: This will take 30-60+ minutes on a Raspberry Pi 4.

4. **Install:**

```bash
sudo make install
```

## Issue 3: Plugin Compatibility Crash

### Problem
After installation, SDR++ Brown crashes with:
```
Thread 1 "sdrpp" received signal SIGSEGV, Segmentation fault.
#1  0x0000007fac107544 in _INIT_ () at /usr/lib/sdrpp/plugins/bladerf_source.so
```

### Root Cause
Old SDR++ plugins from the system package were incompatible with the new SDR++ Brown core library.

### Solution

Remove old plugins and reinstall:

```bash
sudo rm -rf /usr/lib/sdrpp
cd ~/SDRPlusPlusBrown/build
sudo make install
```

## Using CaribouLite with SDR++ Brown

1. **Start SDR++ Brown:**

```bash
sdrpp
```

2. **Configure SoapySDR source:**
   - Click the **Source** dropdown at the top
   - Select **"SoapySDR"**
   - Click the **gear icon** to configure
   - Select **"Cariboulite"** from the device list
   - Click **Start** to begin receiving

3. **Recommended Settings:**
   - **Sample Rate**: Start with 1-2 MSPS
   - **Gain**: Start with 30-40 dB, adjust as needed
   - **Frequencies to try**:
     - 433.92 MHz (ISM band - sensors, weather stations)
     - 446 MHz (PMR446 - walkie talkies in EU)
     - 915 MHz (ISM band in US)

## Features Available

### Built-in Decoders
- **FT8/FT4**: Amateur radio digital modes
- **Meteor M2**: Weather satellite decoder
- **Pager**: POCSAG pager decoding
- **CH Extra VHF**: Swiss emergency services

### Audio Enhancement
- **Noise Reduction**: OMLSA MCRA algorithm for cleaner audio
- **Multiple Audio Sinks**: PulseAudio, PortAudio, Brown Audio Sink

### Network Features
- **Rigctl Server/Client**: CAT control compatibility
- **TCI Server**: Expert Electronics protocol
- **WebSDR View**: Remote SDR viewing
- **Reports Monitor**: Integration with PSKReporter and similar

## Troubleshooting

### SDR++ Brown won't start
Check for conflicting plugins:
```bash
ls -la /usr/lib/sdrpp/plugins/
```

All plugins should have the same modification date from your build.

### CaribouLite not detected
Verify the driver loads:
```bash
SoapySDRUtil --find
```

Should list "Cariboulite" in available factories.

### Audio issues
Check available audio devices:
```bash
aplay -l
pactl list sinks
```

### Build errors
If you get BLAS/LAPACK warnings during cmake, they're safe to ignore - the bundled itpp library will work without them.

## Configuration Files

SDR++ Brown stores configuration in:
```
~/.config/sdrpp-brown/
```

Key files:
- `config.json` - Main application settings
- `soapy_source_config.json` - SoapySDR device settings
- `radio_config.json` - Demodulator settings

## Performance Tips

1. **For Raspberry Pi 4:**
   - Keep FFT size at 65536 or lower
   - Use 1-2 MSPS sample rate for best performance
   - Disable Discord integration if not needed
   - Close unused decoder modules

2. **Reduce CPU usage:**
   - Lower FFT rate (default 20 fps)
   - Disable waterfall if not needed
   - Use narrower filters

## Additional Resources

- **SDR++ Brown GitHub**: https://github.com/sannysanoff/SDRPlusPlusBrown
- **SDR++ Brown Website**: https://sdrpp-brown.san.systems
- **CaribouLite Documentation**: https://cariboulabs.co.il/cariboulite/
- **SoapySDR Documentation**: https://github.com/pothosware/SoapySDR/wiki

## Version Information

This guide was created for:
- **OS**: Raspberry Pi OS (Debian Trixie) 64-bit
- **Kernel**: Linux 6.12.47+rpt-rpi-v8
- **SDR++ Brown**: v1.2.1-brown
- **SoapySDR**: v0.8.1
- **CaribouLite**: SoapyCariboulite v1.2.0

## Credits

- SDR++ by Alexandre Rouma
- SDR++ Brown fork by Sanny Sanoff
- CaribouLite by CaribouLabs
- SoapySDR by Pothosware

## Notes

- The original system SDR++ package conflicts with SDR++ Brown - they cannot coexist
- gqrx remains incompatible with SoapyCariboulite driver as of this writing
- CubicSDR may have similar issues to gqrx
- Other SoapySDR applications (e.g., SoapySDR-Remote) should work fine

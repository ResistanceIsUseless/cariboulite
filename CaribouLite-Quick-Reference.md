# CaribouLite Quick Reference

## Quick Start Commands

### Check if CaribouLite is working
```bash
SoapySDRUtil --probe=driver=Cariboulite
```

### Start SDR++ Brown
```bash
sdrpp
```

### Re-enable CaribouLite for other tools
```bash
sudo mv /usr/lib/aarch64-linux-gnu/SoapySDR/modules0.8/libSoapyCariboulite.so.disabled \
         /usr/lib/aarch64-linux-gnu/SoapySDR/modules0.8/libSoapyCariboulite.so
```

### Disable CaribouLite (if causing crashes in gqrx)
```bash
sudo mv /usr/lib/aarch64-linux-gnu/SoapySDR/modules0.8/libSoapyCariboulite.so \
         /usr/lib/aarch64-linux-gnu/SoapySDR/modules0.8/libSoapyCariboulite.so.disabled
```

## CaribouLite Specifications

| Specification | Value |
|--------------|-------|
| **Model** | CaribouLite S1G |
| **Hardware Rev** | 2.8 |
| **RX Frequency** | 389.5-510 MHz, 779-1020 MHz |
| **TX Frequency** | 389.5-510 MHz, 779-1020 MHz |
| **RX Gain Range** | 0-69 dB |
| **TX Gain Range** | 0-31 dB |
| **Sample Rates** | 0.4, 0.5, 0.667, 0.8, 1, 1.33, 2, 4 MSPS |
| **RX Filters** | 0.02-2 MHz |
| **TX Filters** | 0.08-1 MHz |

## SDR++ Brown Build Commands

```bash
# Install dependencies
sudo apt install -y cmake libad9361-dev libairspy-dev libairspyhf-dev \
    libfftw3-dev libglfw3-dev libhackrf-dev libiio-dev librtaudio-dev \
    libvolk-dev libzstd-dev git build-essential libsoapysdr-dev soapysdr-tools

# Clone and build
git clone https://github.com/sannysanoff/SDRPlusPlusBrown.git
cd SDRPlusPlusBrown
mkdir build && cd build
cmake -DOPT_BUILD_SOAPY_SOURCE=ON ..
make -j1  # Single thread for Raspberry Pi
sudo make install
```

## Troubleshooting One-Liners

### Check loaded SoapySDR modules
```bash
SoapySDRUtil --info
```

### Find all SoapySDR devices
```bash
SoapySDRUtil --find
```

### Check SDR++ plugins
```bash
ls -la /usr/lib/sdrpp/plugins/
```

### View SDR++ Brown config
```bash
ls -la ~/.config/sdrpp-brown/
```

### Check for segfault details
```bash
dmesg | tail -20
```

### Debug SDR++ Brown crash
```bash
gdb -batch -ex "run" -ex "bt" --args sdrpp 2>&1 | tail -50
```

## Interesting Frequencies for CaribouLite S1G

### 389.5-510 MHz Band
- **433.050-434.790 MHz**: ISM band (EU) - Remote controls, sensors, LoRa
- **446.00625-446.19375 MHz**: PMR446 (EU walkie-talkies)
- **462.5-467.7 MHz**: FRS/GMRS (US walkie-talkies) - *Outside range*

### 779-1020 MHz Band
- **868.0-868.6 MHz**: ISM band (EU) - LoRaWAN, sensors
- **915.0-928.0 MHz**: ISM band (US) - LoRa, ZigBee, sensors
- **806-824 MHz**: Public safety, commercial radio
- **851-869 MHz**: Cellular uplink

**Note**: Always check local regulations before transmitting!

## Config File Locations

```
~/.config/sdrpp-brown/config.json                  # Main config
~/.config/sdrpp-brown/soapy_source_config.json    # SoapySDR settings
~/.config/sdrpp-brown/radio_config.json           # Radio/demod settings
~/.config/sdrpp-brown/ft8_decoder_config.json     # FT8 decoder settings
```

## Performance Settings for Raspberry Pi 4

Edit `~/.config/sdrpp-brown/config.json`:

```json
{
    "fftSize": 65536,        // Lower if laggy (32768, 16384)
    "fftRate": 20,           // Lower for less CPU (10-15)
    "source": "SoapySDR",
    "decimation": 1,
    "waterfall": true        // Set false to save CPU
}
```

## Common Issues

| Issue | Solution |
|-------|----------|
| gqrx crashes on startup | Disable SoapyCariboulite module |
| SDR++ Brown crashes | Remove old plugins: `sudo rm -rf /usr/lib/sdrpp && sudo make install` |
| CaribouLite not found | Check driver: `SoapySDRUtil --probe=driver=Cariboulite` |
| No audio output | Check PulseAudio: `pactl list sinks` |
| High CPU usage | Lower FFT size and rate in settings |

## Useful SoapySDR Commands

```bash
# Probe specific device
SoapySDRUtil --probe="driver=Cariboulite"

# Make test application
SoapySDRUtil --make="driver=Cariboulite"

# Check device tree
SoapySDRUtil --find="driver=Cariboulite"

# Rate test (check max sample rate)
SoapySDRUtil --rate="driver=Cariboulite,sampleRate=4e6"
```

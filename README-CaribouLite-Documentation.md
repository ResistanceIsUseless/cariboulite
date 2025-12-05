# CaribouLite Documentation Index

This folder contains complete documentation for setting up and using CaribouLite SDR on Raspberry Pi with modern Linux kernels.

## Files Overview

### Documentation Files

1. **CaribouLite-SDR-Setup-Guide.md** (6.5 KB)
   - Complete setup guide for CaribouLite with SDR software
   - Covers gqrx crash issues and workarounds
   - Step-by-step SDR++ Brown installation
   - Plugin compatibility troubleshooting

2. **CaribouLite-Quick-Reference.md** (4.0 KB)
   - Quick command reference
   - Device specifications
   - Interesting frequencies to monitor
   - Common troubleshooting commands
   - Performance tuning tips

3. **CaribouLite-Kernel-Fixes.md** (5.8 KB)
   - Detailed explanation of kernel driver fixes
   - Kernel 6.4+ and 6.11+ API compatibility
   - Code changes with explanations
   - Build instructions

### Patch Files

4. **cariboulite-driver-changes.patch** (3.8 KB)
   - Git patch for driver changes only
   - Ready to apply with `git apply`
   - Includes CMakeLists.txt and smi_stream_dev.c fixes

5. **cariboulite-changes.patch** (348 KB)
   - Complete git diff of all changes
   - Includes generated code changes
   - Use driver-changes.patch for cleaner application

## Quick Start

### If you need to fix CaribouLite driver on kernel 6.4+:

```bash
cd ~/Downloads/cariboulite
git apply ~/Desktop/cariboulite-driver-changes.patch
cd driver
mkdir build && cd build
cmake ..
make
sudo make install
```

### If you need to install SDR++ Brown:

See **CaribouLite-SDR-Setup-Guide.md** section "Issue 2: Installing SDR++ Brown"

### If gqrx is crashing:

See **CaribouLite-SDR-Setup-Guide.md** section "Issue 1: gqrx Crashes on Startup"

## What Problems Do These Fixes Solve?

### 1. Kernel Driver Build Errors (Kernel 6.4+)
**Symptoms:**
- Build fails with `class_create()` errors
- Build fails with `platform_driver.remove()` type errors

**Solution:** Apply `cariboulite-driver-changes.patch`

### 2. gqrx Segmentation Fault
**Symptoms:**
```
gqrx crashes with: SIGSEGV: memory access violation
CaribouLite: Signal [11] received
```

**Solution:** Disable SoapyCariboulite module for gqrx (see setup guide)

### 3. SDR++ Brown Crashes After Installation
**Symptoms:**
```
Thread 1 "sdrpp" received signal SIGSEGV
at /usr/lib/sdrpp/plugins/bladerf_source.so
```

**Solution:** Remove old plugins, reinstall (see setup guide)

## System Requirements

- **Platform**: Raspberry Pi 4 (tested)
- **OS**: Raspberry Pi OS 64-bit (Debian Trixie/Bookworm)
- **Kernel**: 6.4+ (fixes for 6.12.47 tested)
- **Hardware**: CaribouLite S1G or Full

## Features Working After Setup

✅ CaribouLite driver loads properly
✅ SoapySDR detection and probing
✅ SDR++ Brown with full feature set:
  - Built-in FT8/FT4 decoder
  - Advanced noise reduction
  - Hermes Lite 2 support
  - KiwiSDR integration
  - WebSDR view
  - Reports monitor (PSKReporter)

## Known Issues

❌ **gqrx**: Crashes due to SoapyCariboulite driver bug (upstream issue)
❌ **CubicSDR**: May have similar crash issues (untested)
⚠️ **Old SDR++**: Conflicts with SDR++ Brown (cannot coexist)

## How to Share This Information

All files on the Desktop can be shared together. They reference each other and provide complete documentation.

### For Forum/GitHub Posts:

**Short version:** Share `CaribouLite-Quick-Reference.md`

**Complete version:** Share all 3 markdown files + driver patch

**Code fixes only:** Share `cariboulite-driver-changes.patch`

## File Purposes

| File | Best For |
|------|----------|
| Setup Guide | First-time installation, troubleshooting |
| Quick Reference | Daily use, command lookup |
| Kernel Fixes | Understanding changes, upstream submission |
| driver-changes.patch | Applying fixes to CaribouLite repo |
| changes.patch | Complete backup of all modifications |

## Additional Resources

- **CaribouLite GitHub**: https://github.com/cariboulabs/cariboulite
- **SDR++ Brown GitHub**: https://github.com/sannysanoff/SDRPlusPlusBrown
- **SDR++ Brown Website**: https://sdrpp-brown.san.systems
- **SoapySDR Wiki**: https://github.com/pothosware/SoapySDR/wiki

## Contributing Back

Consider submitting the kernel fixes to the CaribouLite upstream:
1. Fork https://github.com/cariboulabs/cariboulite
2. Apply `cariboulite-driver-changes.patch`
3. Create pull request with subject: "Fix driver build for kernel 6.4+ and 6.11+"

## Version History

- **2024-11-29**: Initial documentation created
  - Kernel 6.12.47 compatibility fixes
  - SDR++ Brown installation guide
  - gqrx workaround documentation

## Credits

Documentation created through collaborative troubleshooting session.

- CaribouLite by CaribouLabs
- SDR++ by Alexandre Rouma
- SDR++ Brown by Sanny Sanoff
- Kernel fixes for modern compatibility

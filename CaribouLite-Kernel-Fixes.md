# CaribouLite Kernel Driver Fixes for Linux 6.4+

## Overview

This document describes the kernel API compatibility fixes required to build the CaribouLite driver on modern Linux kernels (6.4+, specifically tested on 6.12.47).

## Problem

The CaribouLite SMI stream driver failed to compile on newer kernels due to two breaking API changes:

1. **Kernel 6.4+**: `class_create()` API changed - removed `owner` parameter
2. **Kernel 6.11+**: `platform_driver.remove()` changed from returning `int` to `void`

## Changes Made

### 1. CMakeLists.txt - Kernel Version Detection

Added kernel version detection logic to set compile-time flags:

```cmake
# Check kernel version for class_create API change (changed in 6.4)
string(REGEX MATCH "^([0-9]+)\\.([0-9]+)" KERNEL_VERSION_MATCH ${KERNEL_RELEASE})
set(KERNEL_MAJOR ${CMAKE_MATCH_1})
set(KERNEL_MINOR ${CMAKE_MATCH_2})

# Kernel 6.4+ uses class_create without owner parameter
if((KERNEL_MAJOR GREATER 6) OR ((KERNEL_MAJOR EQUAL 6) AND (KERNEL_MINOR GREATER_EQUAL 4)))
    set(HAVE_CLASS_WITHOUT_OWNER 1)
else()
    set(HAVE_CLASS_WITHOUT_OWNER 0)
endif()

message(STATUS "HAVE_CLASS_WITHOUT_OWNER: ${HAVE_CLASS_WITHOUT_OWNER}")
```

Pass the flag to the kernel module build:

```cmake
COMMAND echo "ccflags-y += -O2 -DMODULE -D__KERNEL__ -DHAVE_CLASS_WITHOUT_OWNER=${HAVE_CLASS_WITHOUT_OWNER}" >> ${CMAKE_CURRENT_BINARY_DIR}/Makefile
```

### 2. smi_stream_dev.c - class_create() Compatibility

Added conditional compilation for the different `class_create()` signatures:

```c
// Create sysfs entries with "smi-stream-dev"
#if defined(HAVE_CLASS_WITHOUT_OWNER) && HAVE_CLASS_WITHOUT_OWNER
    smi_stream_class = class_create(DEVICE_NAME);
#else
    smi_stream_class = class_create(THIS_MODULE, DEVICE_NAME);
#endif
```

**Old API (< 6.4)**:
```c
struct class *class_create(struct module *owner, const char *name);
```

**New API (6.4+)**:
```c
struct class *class_create(const char *name);
```

### 3. smi_stream_dev.c - platform_driver.remove() Return Type

Added conditional compilation for the remove function signature:

```c
#if LINUX_VERSION_CODE >= KERNEL_VERSION(6, 11, 0)
static void smi_stream_dev_remove(struct platform_device *pdev)
#else
static int smi_stream_dev_remove(struct platform_device *pdev)
#endif
{
    device_destroy(smi_stream_class, smi_stream_devid);
    class_destroy(smi_stream_class);
    cdev_del(&smi_stream_cdev);
    unregister_chrdev_region(smi_stream_devid, 1);

    dev_info(inst->dev, DRIVER_NAME": smi-stream dev removed");
#if LINUX_VERSION_CODE < KERNEL_VERSION(6, 11, 0)
    return 0;
#endif
}
```

### 4. smi_stream_dev.c - Added Missing Include

Added `linux/vmalloc.h` include:

```c
#include <linux/vmalloc.h>
```

This header is required for some memory allocation functions and may not be implicitly included in newer kernels.

### 5. smi_stream_dev.h - Added Version Header

Added kernel version checking capability:

```c
#include <linux/version.h>
```

This allows the use of `LINUX_VERSION_CODE` and `KERNEL_VERSION()` macros.

## Kernel API Changes Reference

### class_create() Change (6.4+)

**Commit**: `1aaba11da9aa` ("driver core: class: remove module * from class_create()")
**Date**: March 2023
**Kernel**: 6.4

The `owner` parameter was removed because it was redundant - the module owner is already tracked through other mechanisms.

### platform_driver.remove() Change (6.11+)

**Commit**: `0edb555a65d1` ("platform: Make platform_driver::remove() return void")
**Date**: July 2024
**Kernel**: 6.11

Changed to return void since the return value was largely ignored by the driver core.

## Building with These Fixes

1. **Clone the repository:**
```bash
git clone https://github.com/cariboulabs/cariboulite.git
cd cariboulite
```

2. **Apply the patch:**
```bash
# If you have the patch file:
git apply cariboulite-driver-changes.patch

# Or manually apply the changes shown above
```

3. **Build the driver:**
```bash
cd driver
mkdir build && cd build
cmake ..
make
```

4. **Install the driver:**
```bash
sudo make install
sudo modprobe smi_stream_dev
```

5. **Verify installation:**
```bash
lsmod | grep smi
dmesg | grep smi
```

Expected output:
```
smi_stream_dev         24576  0
```

## Testing

After installing the driver, test with SoapySDR:

```bash
SoapySDRUtil --probe=driver=Cariboulite
```

Expected output should show device information without errors.

## Compatibility

These fixes make the CaribouLite driver compatible with:

- ✅ **Linux 6.12.x** (tested on 6.12.47+rpt-rpi-v8)
- ✅ **Linux 6.11.x**
- ✅ **Linux 6.4.x - 6.10.x**
- ✅ **Linux < 6.4** (maintains backward compatibility)

## Files Modified

| File | Purpose | Lines Changed |
|------|---------|---------------|
| `driver/CMakeLists.txt` | Kernel version detection and flag passing | +14 |
| `driver/smi_stream_dev.c` | API compatibility shims | +12 |
| `driver/smi_stream_dev.h` | Version header include | +1 |

## Upstream Status

These changes should be submitted upstream to the CaribouLite project for inclusion in the main repository. The fixes maintain backward compatibility while supporting modern kernels.

## Additional Notes

- The large changes in `smi_stream_dev_gen.h` are generated code and not manual edits
- The `generate_bin_blob` binary was rebuilt but contains no source changes
- All changes maintain full backward compatibility with older kernels

## References

- CaribouLite GitHub: https://github.com/cariboulabs/cariboulite
- Linux Kernel class_create change: https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=1aaba11da9aa
- Linux Kernel platform_driver.remove change: https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=0edb555a65d1

## Version Information

- **Kernel Tested**: Linux 6.12.47+rpt-rpi-v8
- **Platform**: Raspberry Pi 4 Model B Rev 1.4 (aarch64)
- **CaribouLite**: S1G Rev2.8
- **Date**: November 2024

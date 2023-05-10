The main resources in this section are the official piOS building method.
We provide methods to compile locally on Raspberry Pi and cross-compile on PC. It is recommended to use Raspberry Pi local compilation, which is simpler and faster.
# Building the Drivers Locally
The following operations are performed on the Raspberry Pi.
The following is an example of piOS 5.15.32.
## Prepare environment
```
sudo apt install git bc bison flex libssl-dev make
```
## Download the kernel headers for the local piOS
```
sudo apt install raspberrypi-kernel-headers
```
## Download drivers code from veye
```
git clone https://github.com/veyeimaging/raspberrypi_v4l2.git
```
## Confirm the version of piOS on your own Raspberry Pi
```
$ uname -a
Linux raspberrypi 5.15.32-v8+ #1538 SMP PREEMPT Thu Mar 31 19:40:39 BST 2022 aarch64 GNU/Linux
```
## Compile drivers
```
cd ~/raspberrypi_v4l2/driver_source/cam_drv_src/rpi-5.15_all
make
```
Done.

## Compile dts
```
cd ~/raspberrypi_v4l2/driver_source/dts/rpi-5.15.y
./build_dtbo.sh
```
Done.

## Common errors
### Missing build directory
```
make[1]: *** /lib/modules/[version]/build: No such file or directory. Stop.
```
The cmd `sudo apt install raspberrypi-kernel-headers` will simply install the latest kernel headers available on the mirror. Which indeed matches the latest available kernel on the mirror. But not necessarily the installed kernel on the system.

The Raspbian maintainers always remove the older kernel headers from the repository index files. Not sure why they do that.

Solutions：

#### 1. Upgrade piOS, and build again.
`sudo apt update`
`sudo apt full-upgrade`
`sudo apt install raspberrypi-kernel raspberrypi-kernel-headers`

Specifically, for the Raspberry Pi 4 series, 32-bit PiOS will automatically switch to 64-bit mode after upgrading to the latest version. However, the raspberrypi-kernel-headers package is missing the build directory for the v8+ mode.

For example:
```
$ uname -a
Linux raspberrypi 6.1.21-v8+ #1642 SMP PREEMPT Mon Apr 3 17:24:16 BST 2023 aarch64 GNU/Linux
```
There is no build directory under `/lib/modules/6.1.21-v8+/`.

If this happens, you can add `arm_64bit=0` to the `/boot/config.txt` file and then restart the Raspberry Pi to switch back to 32-bit mode.

#### 2. Install specific version kernel headers.

Download the deb package for the version of piOS you are currently using from this link and install it.
https://archive.raspberrypi.org/debian/pool/main/r/raspberrypi-firmware/

For tag name, please determine according to the local piOS version and raspberrypi OS tags.

https://github.com/raspberrypi/linux/tags

#### Use rpi-source
Use rpi-source from https://github.com/RPi-Distro/rpi-source it sets up everything you need to build your own kernel (from the current running kernel by default).

# Cross-Compiling the Drivers
The following operations are done on ubuntu PC.

## Build cross-compilation environment on Ubuntu 64-bit operating system)
```
sudo apt install git bc bison flex libssl-dev make libc6-dev libncurses5-dev
```
- 32-bit piOS target version
```
sudo apt install crossbuild-essential-armhf
```
- 64-bit piOS target version
```
sudo apt install crossbuild-essential-arm64
```
## Download the standard version of the piOS source code
Confirm the piOS version of your raspberry Pi
- Release version
```
$ uname -a
Linux raspberrypi 5.4.72-v7l+ #1356 SMP Thu Oct 22 13:57:51 BST 2020 armv7l GNU/Linux
``` 
- Code tag
```
$ cp /usr/share/doc/raspberrypi-bootloader/changelog.Debian.gz ./
gunzip changelog.Debian.gz
```
Check the top lines and you will know the tag.
### There are two ways to get the source code of the version you want:
1. using git to clone the corresponding branch and checkout the corresponding tag.
```
git clone --branch rpi-5.4.y https://github.com/raspberrypi/linux
git checkout raspberrypi-kernel_1.20201022-1
``` 
PS: Please replace the above two commands with your own corresponding versions.
2. Manually download the version code of the corresponding tag directly from the link below.

https://github.com/raspberrypi/linux/tags

Patch our code to kernel
```
git clone https://github.com/veyeimaging/raspberrypi_v4l2.git
```
- Driver source code
camera driver path is ： linux/drivers/media/i2c，copy our camera module drivers to this path.
-Modify the the Makefile and Kconfig files
Modify the Config and Makefile in the same path，add the corresponding camera driver.
- dts file
dts file path is ：linux/arch/arm/boot/dts/overlays，copy our [camera]-overlay.dts files to this path.
- Modify dts Makefile
Modify the Makefile in the same path，add the corresponding dts compilation option.

## Building
### Preparing
- For Raspberry Pi 1, Zero and Zero W, and Raspberry Pi Compute Module 1 default (32-bit only) build configuration
```
KERNEL=kernel

make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- bcmrpi_defconfig
```
- For Raspberry Pi 2, 3, 3+ and Zero 2 W, and Raspberry Pi Compute Modules 3 and 3+ default 32-bit build configuration
```
KERNEL=kernel7

make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- bcm2709_defconfig
```
- For Raspberry Pi 4 and 400, and Raspberry Pi Compute Module 4 default 32-bit build configuration
```
KERNEL=kernel7l

make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- bcm2711_defconfig
```
- For Raspberry Pi 3, 3+, 4, 400 and Zero 2 W, and Raspberry Pi Compute Modules 3, 3+ and 4 default 64-bit build configuration
```
KERNEL=kernel8

make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- bcm2711_defconfig
```
### Add compilation options
- 32-bit version
```
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- menuconfig
```
- 64-bit version
```
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- menuconfig
```
Add the compilation options by the corresponding camera module, for 5.4 version kernel the path is driver-- > multimedia-- > i2C.

For 5.10 version kernel,the path is Device Drivers --> Multimedia Support --> Media ancillary drivers --> Camera sensor devices.

## Build
- 32-bit version
```
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- zImage modules dtbs -j4
```
- 64-bit version
```
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- Image modules dtbs -j4
```
# References

https://www.raspberrypi.org/documentation/linux/kernel/building.md


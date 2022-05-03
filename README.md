# BPI-M2 Zero kernel

Linux kernel based on the mainline 5.17.5 kernel for [Banana Pi M2 Zero](https://wiki.banana-pi.org/Banana_Pi_BPI-M2_ZERO) with WiFi (and wireguard)

## Build & Install

1. I built this on Ubuntu 22.04 x86\_64 with the [Arm GNU Toolchain](https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain/gnu-a/downloads) (arm-none-linux-gnueabihf), so download and extract it and set the bin dir to PATH:

```
export PATH=$PATH:/opt/gcc-arm-10.3-2021.07-x86_64-arm-none-linux-gnueabihf/bin
```

2. Install the following tools and libs:
```
sudo apt-get install flex bison g++ libgmp3-dev libmpc-dev
```

3. Build from this project root.

```
make INSTALL_MOD_PATH=output ARCH=arm CROSS_COMPILE=arm-none-linux-gnueabihf- m2z_lima_defconfig zImage modules modules_install dtbs -j$(nproc)
```

4. Copy over the output/* (lib/modules) into rootfs of memory card (Follow the [instructions here to install ubuntu](https://github.com/avafinger/bananapi-zero-ubuntu-base-minimal/releases/tag/v3.10) in the memory card):

```
sudo cp -vfr ./output/* /
sync
```

5. Install into the /boot dir of the memory card:

```
export KV=$(strings ./arch/arm/boot/Image |grep "Linux version"|awk '{print $3}')
sudo cp -fv ./arch/arm/boot/zImage /boot/zImage_${KV}
sync
sudo cp -fv ./arch/arm/boot/dts/bpi-m2-zero-v4.dtb /boot/bpi-m2-zero.dtb_${KV}
sync
```

6. Update the symlinks to point to new dtb and zImage inside /boot

```
cd /boot/
sudo ln -sf bpi-m2-zero.dtb_${KV} bpi-m2-zero.dtb
sudo ln -sf zImage_${KV} zImage
```

### Changes from mainline

* Used the m2z_lima_defconfig from https://github.com/avafinger/linux-5.6.y with some modifications to it. ([This](https://github.com/avafinger/bananapi-zero-ubuntu-base-minimal/issues/38#issuecomment-632062680) and some from [here](https://github.com/BPI-SINOVOIP/BPI-M2P-bsp-4.4/blob/b034b7104be40a9fa23a9e8473ef2a1db0d6679c/linux-sunxi/arch/arm/configs/sun8iw7p1smp_bpi-m2z_defconfig#L1821-L1825), although I'm not sure if the latter was necessary for wifi to be working)

* Added [sun8iw7p1smp_bpi-m2z_defconfig](https://github.com/BPI-SINOVOIP/BPI-M2P-bsp-4.4/blob/b034b7104be40a9fa23a9e8473ef2a1db0d6679c/linux-sunxi/arch/arm/configs/sun8iw7p1smp_bpi-m2z_defconfig) and [m2z_defconfig](https://github.com/avafinger/linux-5.6.y/blob/master/arch/arm/configs/m2z_defconfig).

* Added [bpi-m2-zero-v4.dts](https://github.com/avafinger/linux-5.6.y/blob/6dc50035fef1f65b7f5b3b818f69e1e7a3ef0616/arch/arm/boot/dts/bpi-m2-zero-v4.dts), along with the headers it requires and modified the Makefile to accomodate this.

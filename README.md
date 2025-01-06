# Android Automotive 11 for rpi 4

Forked from https://github.com/android-rpi/local_manifests

# Dev env Setup
## Install build components (e.g.: Ubuntu)

```
sudo apt update && sudo apt install gcc-aarch64-linux-gnu libssl-dev bc python3-setuptools repo python-is-python3 libncurses5 zip unzip make gcc flex bison -y
```

## Download Android source
 Refer to http://source.android.com/source/downloading.html
 ```
 mkdir android-11.0.0_r48 && cd android-11.0.0_r48
 repo init -u https://android.googlesource.com/platform/manifest -b android-11.0.0_r48 --partial-clone --clone-filter=blob:limit=10M
```

## Clone local_manifests
 ```
git clone https://github.com/mlorenzati/local_manifests .repo/local_manifests -b arpi-11
 ```

## Sync Repo
```
repo sync
```

# Patching

Just use the patch in https://github.com/mlorenzati/device_arpi_rpi4/blob/arpi-11/aosp_arpi11.patch from root folder
```
patch -p1 < device/arpi/rpi4/aosp_arpi11.patch
```
# Build for Raspberry Pi 4
 Explained above, but for more details go to https://github.com/mlorenzati/device_arpi_rpi4/tree/arpi-11

Use -j[n] option on sync & build steps, if build host has a good number of CPU cores.

# Build Kernel
```
cd kernel/arpi
ARCH=arm64 scripts/kconfig/merge_config.sh arch/arm64/configs/bcm2711_defconfig kernel/configs/android-base.config kernel/configs/android-recommended.config
ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- make Image.gz
ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- DTC_FLAGS="-@" make broadcom/bcm2711-rpi-4-b.dtb
ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- DTC_FLAGS="-@" make overlays/vc4-kms-v3d-pi4.dtbo
cd ../..
```

# Build System
```
source build/envsetup.sh
lunch rpi4-eng
make -j 8 ramdisk systemimage vendorimage
```

# Prepare the SD Image

Follow the steps from below, check your drive in /dev/sdx

```
sudo umount /dev/sdb*
sudo wipefs -a /dev/sdb
sudo fdisk /dev/sdb
```

fdisk steps:

```
Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): 
Using default response p.
Partition number (1-4, default 1): 
First sector (2048-61022207, default 2048): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-61022207, default 61022207): +128M
Created a new partition 1 of type 'Linux' and of size 128 MiB.
Command (m for help): a
Selected partition 1
The bootable flag on partition 1 is enabled now.
Command (m for help): t
Selected partition 1
Hex code or alias (type L to list all): 0c
Changed type of partition 'Linux' to 'W95 FAT32 (LBA)'.
Command (m for help): n
Partition type
   p   primary (1 primary, 0 extended, 3 free)
   e   extended (container for logical partitions)
Select (default p): 
Using default response p.
Partition number (2-4, default 2): 
First sector (264192-61022207, default 264192): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (264192-61022207, default 61022207): +2G
Created a new partition 2 of type 'Linux' and of size 2 GiB.
Command (m for help): n
Partition type
   p   primary (2 primary, 0 extended, 2 free)
   e   extended (container for logical partitions)
Select (default p): 
Using default response p.
Partition number (3,4, default 3): 
First sector (4458496-61022207, default 4458496): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (4458496-61022207, default 61022207): +128M
Created a new partition 3 of type 'Linux' and of size 128 MiB.
Command (m for help): n
Partition type
   p   primary (3 primary, 0 extended, 1 free)
   e   extended (container for logical partitions)
Select (default e): p
Selected partition 4
First sector (4720640-61022207, default 4720640): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (4720640-61022207, default 61022207):
Created a new partition 4 of type 'Linux' and of size 26,8 GiB.
Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```

2.Create the filesystem:
```
sudo mkdosfs -F 32 /dev/sdb1
sudo mkfs.ext4 -L userdata /dev/sdb4
```

3.Copy the artifacts
```
sudo mkdir /mnt/p1
sudo mount /dev/sdb1 /mnt/p1
sudo mkdir /mnt/p1/overlays
cd <PATH_TO_YOUR_ANDROID_SOURCES_IN_WSL>
sudo cp device/arpi/rpi4/boot/* /mnt/p1
sudo cp kernel/arpi/arch/arm64/boot/Image.gz /mnt/p1
sudo cp kernel/arpi/arch/arm64/boot/dts/broadcom/bcm2711-rpi-4-b.dtb /mnt/p1
sudo cp kernel/arpi/arch/arm/boot/dts/overlays/vc4-kms-v3d-pi4.dtbo /mnt/p1/overlays/
sudo cp out/target/product/rpi4/ramdisk.img /mnt/p1
sudo umount /mnt/p1
sudo rm -rf /mnt/p1
```

4.DD the images
```
cd <PATH_TO_YOUR_ANDROID_SOURCES>/out/target/product/rpi4/
sudo dd if=system.img of=/dev/sdb2 bs=1M status=progress
sudo dd if=vendor.img of=/dev/sdb3 bs=1M status=progress
```


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

# Build for Raspberry Pi 4
 https://github.com/android-rpi/device_arpi_rpi4/tree/arpi-11

Use -j[n] option on sync & build steps, if build host has a good number of CPU cores.

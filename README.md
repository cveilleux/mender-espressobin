# What

Demo project for Yocto + Mender + Espressobin

Discussion/announcement:

https://hub.mender.io/t/preliminary-support-for-marvell-globalscale-espressobin-v5-v7/4538



# Requirements

- A supported host system with build tools for yocto:
  - https://www.yoctoproject.org/docs/current/mega-manual/mega-manual.html#detailed-supported-distros
- Google repo tool:
  - https://hub.mender.io/t/google-repo/58

# Project setup

```
mkdir espressobin
cd espressobin
repo init -u https://github.com/cveilleux/mender-espressobin.git -m manifest.xml -b master
repo sync
source sources/poky/oe-init-build-env build/
```

At this point you should be in an fresh "build" directory, with the following files initialized:

```
.
└── conf
    ├── bblayers.conf
    ├── local.conf
    └── templateconf.cfg
```

Enable the layers:

```
bitbake-layers add-layer ../sources/meta-espressobin/ ../sources/meta-mender/meta-mender-core/ ../sources/meta-mender-espressobin/
```


Edit the local.conf and append the following for mender at the end of the file:

```
MENDER_ARTIFACT_NAME = "release-1"

INHERIT += "mender-full"

DISTRO_FEATURES_append = " systemd"
VIRTUAL-RUNTIME_init_manager = "systemd"
DISTRO_FEATURES_BACKFILL_CONSIDERED = "sysvinit"
VIRTUAL-RUNTIME_initscripts = ""

MACHINE ?= "espressobin-v7"

IMAGE_INSTALL_append = " kernel-image kernel-devicetree"

MENDER_IMAGE_BOOTLOADER_FILE = "u-boot-${MACHINE}.bin"

MENDER_BOOT_PART_SIZE_MB = "32"

MENDER_FEATURES_ENABLE_append = " mender-uboot mender-image-sd"
MENDER_FEATURES_DISABLE_append = " mender-grub mender-image-uefi"
```

Note: You might want to change `MACHINE` to `espressobin-v5` depending on your board version.


# Build

```
bitbake core-image-minimal
```

## Results

If the build succeeds, the deployable artefacts will be located under `build/tmp/deploy/images/${MACHINE}`.

For example:

```
yocto@build1:~/git/espressobin/build$ ls -lh tmp/deploy/images/espressobin-v5/
total 191M
-rw-r--r-- 3 yocto yocto   12K Jan 26 18:59 armada-3720-espressobin-v5-emmc--5.10.14-r0-espressobin-v5-20220126184327.dtb
lrwxrwxrwx 2 yocto yocto    77 Jan 26 18:59 armada-3720-espressobin-v5-emmc.dtb -> armada-3720-espressobin-v5-emmc--5.10.14-r0-espressobin-v5-20220126184327.dtb
lrwxrwxrwx 2 yocto yocto    77 Jan 26 18:59 armada-3720-espressobin-v5-emmc-espressobin-v5.dtb -> armada-3720-espressobin-v5-emmc--5.10.14-r0-espressobin-v5-20220126184327.dtb
-rw-rw-r-- 2 yocto yocto  4.5K Jan 26 19:20 core-image-minimal.env
-rw-r--r-- 2 yocto yocto  128M Jan 26 19:20 core-image-minimal-espressobin-v5-20220126191935.dataimg
-rw-r--r-- 2 yocto yocto  416M Jan 26 19:20 core-image-minimal-espressobin-v5-20220126191935.ext4
-rw-r--r-- 2 yocto yocto  2.3K Jan 26 19:20 core-image-minimal-espressobin-v5-20220126191935.manifest
-rw-rw-r-- 2 yocto yocto   30M Jan 26 19:20 core-image-minimal-espressobin-v5-20220126191935.mender
-rw-r--r-- 2 yocto yocto  2.3K Jan 26 19:20 core-image-minimal-espressobin-v5-20220126191935.mender.bmap
-rw-r--r-- 2 yocto yocto 1016M Jan 26 19:20 core-image-minimal-espressobin-v5-20220126191935.sdimg
-rw-r--r-- 2 yocto yocto  6.2K Jan 26 19:20 core-image-minimal-espressobin-v5-20220126191935.sdimg.bmap
-rw-r--r-- 2 yocto yocto   33M Jan 26 19:20 core-image-minimal-espressobin-v5-20220126191935.sdimg.bz2
-rw-rw-r-- 2 yocto yocto   25M Jan 26 19:20 core-image-minimal-espressobin-v5-20220126191935.tar.bz2
-rw-r--r-- 2 yocto yocto  312K Jan 26 19:20 core-image-minimal-espressobin-v5-20220126191935.testdata.json
lrwxrwxrwx 2 yocto yocto    56 Jan 26 19:20 core-image-minimal-espressobin-v5.dataimg -> core-image-minimal-espressobin-v5-20220126191935.dataimg
lrwxrwxrwx 2 yocto yocto    53 Jan 26 19:20 core-image-minimal-espressobin-v5.ext4 -> core-image-minimal-espressobin-v5-20220126191935.ext4
lrwxrwxrwx 2 yocto yocto    57 Jan 26 19:20 core-image-minimal-espressobin-v5.manifest -> core-image-minimal-espressobin-v5-20220126191935.manifest
lrwxrwxrwx 2 yocto yocto    55 Jan 26 19:20 core-image-minimal-espressobin-v5.mender -> core-image-minimal-espressobin-v5-20220126191935.mender
lrwxrwxrwx 2 yocto yocto    60 Jan 26 19:20 core-image-minimal-espressobin-v5.mender.bmap -> core-image-minimal-espressobin-v5-20220126191935.mender.bmap
lrwxrwxrwx 2 yocto yocto    54 Jan 26 19:20 core-image-minimal-espressobin-v5.sdimg -> core-image-minimal-espressobin-v5-20220126191935.sdimg
lrwxrwxrwx 2 yocto yocto    59 Jan 26 19:20 core-image-minimal-espressobin-v5.sdimg.bmap -> core-image-minimal-espressobin-v5-20220126191935.sdimg.bmap
lrwxrwxrwx 2 yocto yocto    58 Jan 26 19:20 core-image-minimal-espressobin-v5.sdimg.bz2 -> core-image-minimal-espressobin-v5-20220126191935.sdimg.bz2
lrwxrwxrwx 2 yocto yocto    56 Jan 26 19:20 core-image-minimal-espressobin-v5.tar.bz2 -> core-image-minimal-espressobin-v5-20220126191935.tar.bz2
lrwxrwxrwx 2 yocto yocto    62 Jan 26 19:20 core-image-minimal-espressobin-v5.testdata.json -> core-image-minimal-espressobin-v5-20220126191935.testdata.json
-rw-r--r-- 2 yocto yocto    61 Jan 26 19:02 fw_env.config.default
lrwxrwxrwx 2 yocto yocto    51 Jan 26 18:59 Image -> Image--5.10.14-r0-espressobin-v5-20220126184327.bin
-rw-r--r-- 3 yocto yocto   18M Jan 26 18:59 Image--5.10.14-r0-espressobin-v5-20220126184327.bin
lrwxrwxrwx 2 yocto yocto    51 Jan 26 18:59 Image-espressobin-v5.bin -> Image--5.10.14-r0-espressobin-v5-20220126184327.bin
-rw-rw-r-- 2 yocto yocto  3.0M Jan 26 18:59 modules--5.10.14-r0-espressobin-v5-20220126184327.tgz
lrwxrwxrwx 2 yocto yocto    53 Jan 26 18:59 modules-espressobin-v5.tgz -> modules--5.10.14-r0-espressobin-v5-20220126184327.tgz
lrwxrwxrwx 2 yocto yocto    36 Jan 26 19:02 u-boot.bin -> u-boot-espressobin-v5-2018.03-r0.bin
-rw-r--r-- 2 yocto yocto   16M Jan 26 19:02 uboot.env
lrwxrwxrwx 2 yocto yocto    56 Jan 26 19:02 u-boot-espressobin-initial-env -> u-boot-espressobin-initial-env-espressobin-v5-2018.03-r0
lrwxrwxrwx 2 yocto yocto    56 Jan 26 19:02 u-boot-espressobin-initial-env-espressobin-v5 -> u-boot-espressobin-initial-env-espressobin-v5-2018.03-r0
-rw-r--r-- 2 yocto yocto  2.1K Jan 26 19:02 u-boot-espressobin-initial-env-espressobin-v5-2018.03-r0
-rw-r--r-- 2 yocto yocto  595K Jan 26 19:02 u-boot-espressobin-v5-2018.03-r0.bin
lrwxrwxrwx 2 yocto yocto    36 Jan 26 19:02 u-boot-espressobin-v5.bin -> u-boot-espressobin-v5-2018.03-r0.bin
```

There are 3 interesting files in there:

- `u-boot-espressobin-v5-2018.03-r0.bin`: The uboot binary that must be flashed to replace the stock bootloader.
- `core-image-minimal-espressobin-v5.sdimg`: The sd-card image you can flash to an SD card.
- `core-image-minimal-espressobin-v5.mender`: The mender update file. You can use it to update a running system.


# Flashing

## Boot loader

The stock u-boot boot-loader must be replaced.

There are a few methods outlined here:

http://wiki.espressobin.net/tiki-index.php?page=Update+the+Bootloader

The quickest/simplest is with a USB stick or SD card.

## SD card

To prepare a new SD card for the first time.

Use a tool such as Balena Etcher and flash your card.

## Mender

Once you have a running system with network access.

You can do the OTA update:

```
mender -install http://example.org/update-file.mender
reboot
```

After the reboot, if everything works:

```
mender -commit
```

Just reboot or power-cycle to revert to the previous version before committing.




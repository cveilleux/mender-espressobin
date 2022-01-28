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

MENDER_FEATURES_ENABLE_append = " mender-uboot mender-image-sd"
MENDER_FEATURES_DISABLE_append = " mender-grub mender-image-uefi"
```


# Build

```
bitbake core-image-minimal
```

# Flashing

The build process will produce 3 artefacts:

- The .mender update file you can use to update a running system.
- The .sdimg sd-card image you can flash to a SD card.
- The uboot-img that must be flashed to replace the stock bootloader.

## Boot loader

The stock u-boot boot-loader must be replaced.

There are a few methods outlined here:

http://wiki.espressobin.net/tiki-index.php?page=Update+the+Bootloader

The quickest/simplest is with a USB stick or SD card.

## SD card

To prepare a new SD card for the first time.

Use a tool such as Baleina Etcher and flash your card.

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




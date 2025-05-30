# SPDX-License-Identifier: GPL-2.0-only
#
# Copyright (C) 2010-2016 OpenWrt.org

# boards missing since devicetree update
#EASY50712 ARV3527P

KERNEL_LOADADDR = 0x80002000
KERNEL_ENTRY = 0x80002000

include $(TOPDIR)/rules.mk
include $(INCLUDE_DIR)/image.mk

ifeq ($(SUBTARGET),xway)
	UBIFS_OPTS := -m 512 -e 15872 -c 1959
else
	UBIFS_OPTS := -m 2048 -e 126KiB -c 4096
endif

define Build/append-avm-fakeroot
	cat ./eva.dummy.squashfs >> $@
endef

define Build/dgn3500-sercom-footer
	$(STAGING_DIR_HOST)/bin/dgn3500sum $@ $(1) $(2)
endef

define Build/mkbrncmdline
	mkbrncmdline -i $@ -o $@.new BRN-BOOT
	mv $@.new $@
endef

define Build/mkbrnimg
	mkbrnimg -s $(SIGNATURE) -m $(MAGIC) -p $(CRC32_POLY) -o $@ $(IMAGE_KERNEL) $(IMAGE_ROOTFS)
endef

define Build/fullimage
	mkimage -A mips -O linux -C lzma -T filesystem -a 0x00  \
		-e 0x00 -n '$(VERSION_DIST) RootFS' \
		-d $(IMAGE_ROOTFS) $(IMAGE_ROOTFS).new

	cat $(IMAGE_KERNEL) $(IMAGE_ROOTFS).new > $@.tmp

	mkimage -A mips -O linux -T multi -a 0x00 -C none \
		-e 0x00 -n 'OpenWrt fullimage' \
		-d $@.tmp $@

	rm $(IMAGE_ROOTFS).new
	rm $@.tmp
endef

define Build/loader-common
	rm -rf $@.src
	$(MAKE) -C lzma-loader \
		PKG_BUILD_DIR="$@.src" \
		TARGET_DIR="$(dir $@)" LOADER_NAME="$(notdir $@)" \
		BOARD="$(BOARDNAME)" PLATFORM="lantiq" \
		LZMA_TEXT_START=0x82000000 \
		LOADADDR=0x80002000 \
		$(1) compile loader.bin
	mv "$@.bin" "$@"
	rm -rf $@.src
endef

define Build/loader-kernel
	$(call Build/loader-common,LOADER_DATA="$@")
endef

define Build/loader-okli-compile
	$(call Build/loader-common, \
		FLASH_START=0x10000000 \
		FLASH_OFFS=$(LOADER_FLASH_OFFS) \
		FLASH_MAX=0x0 \
	)
endef

define Build/prepend-loader-okli
	cat "$(KDIR)/loader-$(word 1,$(1)).bin" "$@" >> "$@.new"
	mv "$@.new" "$@"
endef

DEVICE_VARS += SIGNATURE MAGIC CRC32_POLY LOADER_FLASH_OFFS

DTS_DIR := $(DTS_DIR)/lantiq

# Shared device definition: applies to every defined device
define Device/Default
  PROFILES = Default
  COMPILE :=
  KERNEL_DEPENDS = $$(wildcard $(DTS_DIR)/$$(DEVICE_DTS).dts)
  KERNEL_INITRAMFS_NAME = $$(KERNEL_NAME)-initramfs
  KERNEL := kernel-bin | append-dtb | lzma | uImage lzma
  KERNEL_INITRAMFS := kernel-bin | append-dtb | lzma | uImage lzma
  FILESYSTEMS := squashfs
  SOC := $(DEFAULT_SOC)
  DEVICE_DTS = $$(SOC)_$(1)
  IMAGES := sysupgrade.bin
  IMAGE/sysupgrade.bin := append-kernel | append-rootfs | pad-rootfs | check-size | append-metadata
endef

define Device/NAND/xway
  BLOCKSIZE := 16k
  PAGESIZE := 512
  SUBPAGESIZE := 256
  FILESYSTEMS += ubifs
endef

define Device/NAND/xrx200
  BLOCKSIZE := 128k
  PAGESIZE := 2048
  SUBPAGESIZE := 512
  FILESYSTEMS += ubifs
endef

define Device/NAND
  $(Device/NAND/$(SUBTARGET))
  IMAGE/sysupgrade.bin := sysupgrade-tar | append-metadata
endef

define Device/lantiqBrnImage
  KERNEL := kernel-bin | append-dtb | mkbrncmdline | lzma-no-dict
  IMAGES := factory.bin
  IMAGE/factory.bin := mkbrnimg | check-size
endef

define Device/lantiqFullImage
  KERNEL := kernel-bin | append-dtb | lzma | uImage lzma | pad-offset 4 0
  IMAGES := sysupgrade.bin fullimage.bin
  IMAGE/fullimage.bin := fullimage | check-size
endef

define Device/AVM
  DEVICE_VENDOR := AVM
  KERNEL := kernel-bin | append-dtb | lzma | eva-image
  KERNEL_INITRAMFS := $$(KERNEL)
  IMAGE/sysupgrade.bin := append-kernel | pad-to 64k | append-avm-fakeroot | \
	append-rootfs | pad-rootfs | check-size | append-metadata
endef

define Device/AVM_preloader
  DEVICE_VENDOR := AVM
  COMPILE := loader-$(1).bin
  COMPILE/loader-$(1).bin := loader-okli-compile | lzma | eva-image | \
	pad-to 64k | append-avm-fakeroot | pad-to 4k
  KERNEL := kernel-bin | append-dtb | lzma | uImage lzma -M 0x4f4b4c49 | \
	prepend-loader-okli $(1)
  KERNEL_INITRAMFS := kernel-bin | append-dtb | lzma | eva-image
endef

ifeq ($(SUBTARGET),ase)
DEFAULT_SOC := amazonse
include amazonse.mk
endif

ifeq ($(SUBTARGET),xway_legacy)
DEFAULT_SOC := danube
include xway_legacy.mk
endif

ifeq ($(SUBTARGET),xway)
include danube.mk
include ar9.mk
endif

ifeq ($(SUBTARGET),xrx200_legacy)
DEFAULT_SOC := vr9
include tp-link_legacy.mk
include vr9_legacy.mk
endif

ifeq ($(SUBTARGET),xrx200)
DEFAULT_SOC := vr9
include tp-link.mk
include vr9.mk
endif

ifeq ($(SUBTARGET),falcon)
DEFAULT_SOC := falcon
include falcon.mk
endif

$(eval $(call BuildImage))

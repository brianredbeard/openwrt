#
# Copyright (C) 2006-2015 OpenWrt.org
#
# This is free software, licensed under the GNU General Public License v2.
# See /LICENSE for more information.
#
include $(TOPDIR)/rules.mk

PKG_NAME:=xz
PKG_VERSION:=5.8.3

PKG_SOURCE:=$(PKG_NAME)-$(PKG_VERSION).tar.bz2
PKG_SOURCE_URL:=https://github.com/tukaani-project/xz/releases/download/v$(PKG_VERSION) \
		@SF/lzmautils \
		http://tukaani.org/xz
PKG_HASH:=33bf69c0d6c698e83a68f77e6c1f465778e418ca0b3d59860d3ab446f4ac99a6
PKG_CPE_ID:=cpe:/a:tukaani:xz

HOST_BUILD_PARALLEL:=1

include $(INCLUDE_DIR)/host-build.mk

HOSTCC := $(HOSTCC_NOCACHE)
HOSTCXX := $(HOSTCXX_NOCACHE)

HOST_CONFIGURE_ARGS += \
	--enable-static=yes \
	--enable-shared=no \
	--disable-doc \
	--disable-nls \
	--with-pic

# xz 5.8.3 configure's AC_TYPE_INT32_T et al. fail when the host-build
# framework injects -I$(STAGING_DIR_HOST)/include, which disrupts GCC 15's
# C23-conformant header resolution.  Pre-seed the cache so configure skips
# these broken autoconf checks.
HOST_CONFIGURE_VARS += \
	ac_cv_c_int32_t=yes \
	ac_cv_c_int64_t=yes \
	ac_cv_c_uint16_t=yes \
	ac_cv_c_uint32_t=yes \
	ac_cv_c_uint64_t=yes \
	ac_cv_sizeof_size_t=$(if $(filter x86_64 aarch64,$(HOST_ARCH)),8,4)

define Host/Install
	+$(MAKE) $(HOST_JOBS) -C $(HOST_BUILD_DIR) install xzlinks="unxz xzcat"
endef

define Host/Uninstall
	-$(call Host/Compile/Default,uninstall)
endef

$(eval $(call HostBuild))

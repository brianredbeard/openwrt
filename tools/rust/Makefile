# SPDX-License-Identifier: GPL-2.0-only
#
# Copyright (C) 2023 Luca Barbato and Donald Hoskins

include $(TOPDIR)/rules.mk

PKG_NAME:=rust
PKG_VERSION:=1.94.0
PKG_RELEASE:=1

PKG_LICENSE:=Apache-2.0 MIT
PKG_LICENSE_FILES:=LICENSE-APACHE LICENSE-MIT
PKG_CPE_ID:=cpe:/a:rust-lang:rust

include $(INCLUDE_DIR)/rust-values.mk

# Rust-lang has an uninstall script
RUST_UNINSTALL:=$(STAGING_DIR_HOST)/lib/rustlib/uninstall.sh

define Host/Uninstall
	[ -f $(RUST_UNINSTALL) ] && \
		$(BASH) $(RUST_UNINSTALL) || echo No Uninstall
endef

# ---------------------------------------------------------------------------
# Mode 1: Build from source
# ---------------------------------------------------------------------------
ifdef CONFIG_USE_RUST_BUILD

PKG_SOURCE:=rustc-$(PKG_VERSION)-src.tar.xz
PKG_SOURCE_URL:=https://static.rust-lang.org/dist/
PKG_HASH:=0b53ae34f5c0c3612cfe1de139f9167a018cd5737bc2205664fd69ba9b25f600

# Must be set BEFORE host-build.mk is included. quilt.mk does
# Host/Patch:=$(Host/Patch/Default) — an immediate := assignment that captures
# $(HOST_BUILD_DIR) at include time. Setting it first ensures the patch step
# uses the correct directory rather than the ?= default ("rust-1.94.0").
HOST_BUILD_DIR:=$(BUILD_DIR_HOST)/rustc-$(PKG_VERSION)-src

include $(INCLUDE_DIR)/host-build.mk

HOST_CONFIGURE_VARS += CARGO_HOME="$(CARGO_HOME)"

# Build the host compiler only (no cross rust-std here; libstd-rust handles that).
HOST_CONFIGURE_ARGS = \
	--build=$(RUSTC_HOST_ARCH) \
	--target=$(RUSTC_HOST_ARCH) \
	--host=$(RUSTC_HOST_ARCH) \
	--prefix=$(STAGING_DIR_HOST) \
	--bindir=$(STAGING_DIR_HOST)/bin \
	--libdir=$(STAGING_DIR_HOST)/lib \
	--sysconfdir=$(STAGING_DIR_HOST)/etc \
	--datadir=$(STAGING_DIR_HOST)/share \
	--mandir=$(STAGING_DIR_HOST)/man \
	--dist-compression-formats=gz \
	--disable-sanitizers \
	--release-channel=stable \
	--enable-cargo-native-static \
	--bootstrap-cache-path=$(DL_DIR)/rustc \
	"--set=llvm.targets=$(RUST_LLVM_TARGETS)" \
	"--set=llvm.experimental-targets=$(RUST_LLVM_EXPERIMENTAL_TARGETS)" \
	--set=llvm.download-ci-llvm=false

define Host/Compile
	$(RUST_SCCACHE_VARS) \
	CARGO_HOME=$(CARGO_HOME) \
	$(PYTHON) $(HOST_BUILD_DIR)/x.py \
		--build-dir $(HOST_BUILD_DIR)/build \
		dist rustc cargo llvm-tools rust-src
endef

define Host/Install
	( \
		cd $(HOST_BUILD_DIR)/build/dist ; \
		for targz in rustc-* cargo-* llvm-tools-* rust-src-*.tar.gz; do \
			[ -f "$$targz" ] || continue; \
			$(STAGING_DIR_HOST)/bin/libdeflate-gzip -dc "$$targz" | tar -xf - ; \
		done ; \
		find . -mindepth 2 -maxdepth 2 -type f -name install.sh \
			-execdir bash '{}' --prefix=$(STAGING_DIR_HOST) --disable-ldconfig \; ; \
	)
	# Install bootstrap binary so package/libs/libstd-rust can rebuild cross
	# rust-std in the SDK without needing the source build directory.
	$(INSTALL_DIR) $(STAGING_DIR_HOST)/lib/rustlib/bootstrap
	$(INSTALL_BIN) $(HOST_BUILD_DIR)/build/bootstrap/debug/bootstrap \
		$(STAGING_DIR_HOST)/lib/rustlib/bootstrap/
	# Install minimal source metadata into the rust-src root so that bootstrap
	# can determine channel/version and include license files in dist tarballs.
	$(INSTALL_DIR) $(STAGING_DIR_HOST)/lib/rustlib/src/rust/src/ci
	$(INSTALL_DATA) $(HOST_BUILD_DIR)/src/ci/channel \
		$(STAGING_DIR_HOST)/lib/rustlib/src/rust/src/ci/channel
	$(INSTALL_DATA) $(HOST_BUILD_DIR)/src/version \
		$(STAGING_DIR_HOST)/lib/rustlib/src/rust/src/version
	$(CP) $(HOST_BUILD_DIR)/COPYRIGHT \
		$(STAGING_DIR_HOST)/lib/rustlib/src/rust/COPYRIGHT
	$(CP) $(HOST_BUILD_DIR)/LICENSE-APACHE \
		$(STAGING_DIR_HOST)/lib/rustlib/src/rust/LICENSE-APACHE
	$(CP) $(HOST_BUILD_DIR)/LICENSE-MIT \
		$(STAGING_DIR_HOST)/lib/rustlib/src/rust/LICENSE-MIT
endef

endif # CONFIG_USE_RUST_BUILD

# ---------------------------------------------------------------------------
# Mode 2: Download prebuilt components from static.rust-lang.org/dist/
# ---------------------------------------------------------------------------
ifdef CONFIG_USE_RUST_PREBUILT

HOST_BUILD_DIR:=$(BUILD_DIR_HOST)/rust-$(PKG_VERSION)-prebuilt

include $(INCLUDE_DIR)/host-build.mk

# Per-architecture hashes for prebuilt component tarballs.
ifeq ($(HOST_ARCH),x86_64)
  RUSTC_PREBUILT_HASH:=31a0d3ac9383dfdeb4fce86eeed5ade3230131c635264c0eab7252dbf235f28e
  CARGO_PREBUILT_HASH:=8e17624f3de39e079845bfb25ed15a042f4b50ceca78e37c56c4b9b15949b9f7
  LLVM_TOOLS_PREBUILT_HASH:=cf6b8e1d3e48ffa4db37aa59f55382b991f0d347d9b62eebc9ef1d80dffabef4
else ifeq ($(HOST_ARCH),aarch64)
  RUSTC_PREBUILT_HASH:=7261b2fcdb88aef8b21f4b9f915d9a75c959674193f5b38e13fab4569684fc5b
  CARGO_PREBUILT_HASH:=d57ab7b23b52cba46e58c78327904d0c7db19892988677b0d59495c8af4bb790
  LLVM_TOOLS_PREBUILT_HASH:=d9780dbef8b58bf544b255785a420aec64378bc496adb770a628670a0f18171c
endif
RUST_SRC_PREBUILT_HASH:=9412e2f7ad3aac30ad895006d5ef4ce05d044fed48141a41472936d1e1f63cb7

define Download/rustc-prebuilt
  FILE:=rustc-$(PKG_VERSION)-$(RUSTC_HOST_ARCH).tar.xz
  URL:=https://static.rust-lang.org/dist/
  HASH:=$(RUSTC_PREBUILT_HASH)
endef
$(eval $(call Download,rustc-prebuilt))

define Download/cargo-prebuilt
  FILE:=cargo-$(PKG_VERSION)-$(RUSTC_HOST_ARCH).tar.xz
  URL:=https://static.rust-lang.org/dist/
  HASH:=$(CARGO_PREBUILT_HASH)
endef
$(eval $(call Download,cargo-prebuilt))

define Download/rust-src-prebuilt
  FILE:=rust-src-$(PKG_VERSION).tar.xz
  URL:=https://static.rust-lang.org/dist/
  HASH:=$(RUST_SRC_PREBUILT_HASH)
endef
$(eval $(call Download,rust-src-prebuilt))

define Download/llvm-tools-prebuilt
  FILE:=llvm-tools-$(PKG_VERSION)-$(RUSTC_HOST_ARCH).tar.xz
  URL:=https://static.rust-lang.org/dist/
  HASH:=$(LLVM_TOOLS_PREBUILT_HASH)
endef
$(eval $(call Download,llvm-tools-prebuilt))

# Also download the source tarball — only the bootstrap binary is compiled
# from it (minutes, not hours).  libstd-rust needs bootstrap to cross-compile
# rust-std for the target architecture.
define Download/rust-src-full
  FILE:=rustc-$(PKG_VERSION)-src.tar.xz
  URL:=https://static.rust-lang.org/dist/
  HASH:=0b53ae34f5c0c3612cfe1de139f9167a018cd5737bc2205664fd69ba9b25f600
endef
$(eval $(call Download,rust-src-full))

define Host/Prepare
	mkdir -p $(HOST_BUILD_DIR)
	# Install prebuilt component tarballs first — cargo is needed to
	# compile the bootstrap binary in Host/Compile.
	cd $(HOST_BUILD_DIR) && \
	for xz in \
		$(DL_DIR)/rustc-$(PKG_VERSION)-$(RUSTC_HOST_ARCH).tar.xz \
		$(DL_DIR)/cargo-$(PKG_VERSION)-$(RUSTC_HOST_ARCH).tar.xz \
		$(DL_DIR)/llvm-tools-$(PKG_VERSION)-$(RUSTC_HOST_ARCH).tar.xz \
		$(DL_DIR)/rust-src-$(PKG_VERSION).tar.xz; do \
		[ -f "$$xz" ] || continue; \
		tar -xf "$$xz"; \
	done && \
	find . -mindepth 2 -maxdepth 2 -type f -name install.sh \
		-execdir bash '{}' --prefix=$(STAGING_DIR_HOST) --disable-ldconfig \;
	# Extract only the bootstrap crate from the full source.
	tar -xf $(DL_DIR)/rustc-$(PKG_VERSION)-src.tar.xz \
		-C $(HOST_BUILD_DIR) --strip-components=1 \
		rustc-$(PKG_VERSION)-src/src/bootstrap \
		rustc-$(PKG_VERSION)-src/src/ci/channel \
		rustc-$(PKG_VERSION)-src/src/version \
		rustc-$(PKG_VERSION)-src/COPYRIGHT \
		rustc-$(PKG_VERSION)-src/LICENSE-APACHE \
		rustc-$(PKG_VERSION)-src/LICENSE-MIT \
		rustc-$(PKG_VERSION)-src/Cargo.lock
endef

define Host/Configure
	:
endef

define Host/Compile
	# Build only the bootstrap binary using the prebuilt rustc/cargo.
	# This is fast (~1-2 minutes) compared to a full x.py build.
	cd $(HOST_BUILD_DIR)/src/bootstrap && \
	CARGO_HOME=$(CARGO_HOME) \
	$(STAGING_DIR_HOST)/bin/cargo build \
		--release --locked \
		--manifest-path $(HOST_BUILD_DIR)/src/bootstrap/Cargo.toml
endef

define Host/Install
	# Install bootstrap binary for libstd-rust cross-std builds.
	$(INSTALL_DIR) $(STAGING_DIR_HOST)/lib/rustlib/bootstrap
	$(INSTALL_BIN) $(HOST_BUILD_DIR)/src/bootstrap/target/release/bootstrap \
		$(STAGING_DIR_HOST)/lib/rustlib/bootstrap/
	# Install source metadata that bootstrap needs for channel/version.
	$(INSTALL_DIR) $(STAGING_DIR_HOST)/lib/rustlib/src/rust/src/ci
	$(INSTALL_DATA) $(HOST_BUILD_DIR)/src/ci/channel \
		$(STAGING_DIR_HOST)/lib/rustlib/src/rust/src/ci/channel
	$(INSTALL_DATA) $(HOST_BUILD_DIR)/src/version \
		$(STAGING_DIR_HOST)/lib/rustlib/src/rust/src/version
	$(CP) $(HOST_BUILD_DIR)/COPYRIGHT \
		$(STAGING_DIR_HOST)/lib/rustlib/src/rust/COPYRIGHT
	$(CP) $(HOST_BUILD_DIR)/LICENSE-APACHE \
		$(STAGING_DIR_HOST)/lib/rustlib/src/rust/LICENSE-APACHE
	$(CP) $(HOST_BUILD_DIR)/LICENSE-MIT \
		$(STAGING_DIR_HOST)/lib/rustlib/src/rust/LICENSE-MIT
endef

endif # CONFIG_USE_RUST_PREBUILT

$(eval $(call HostBuild))

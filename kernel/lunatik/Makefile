#
# Copyright (C) 2025 OpenWrt.org
#
# This is free software, licensed under the GNU General Public License v2.
# See /LICENSE for more information.
#
include $(TOPDIR)/rules.mk
include $(INCLUDE_DIR)/kernel.mk
include $(INCLUDE_DIR)/netfilter.mk

PKG_NAME:=lunatik
PKG_VERSION:=3.5
PKG_RELEASE:=1.0
PKG_LICENSE:=GPL-2.0

#source code from branch marcel_openwrt
PKG_SOURCE_URL:=https://github.com/luainkernel/lunatik
PKG_SOURCE_PROTO:=git
PKG_SOURCE_DATE:=2025-01-30
PKG_SOURCE_VERSION:=4ac8b93aa4f5e82d81e28df3f9a75d7b8a0cab89

PKG_MAINTAINER:=Lourival Vieira Neto<lourival.neto@gmail.com>
PKG_BUILD_PARALLEL:=1
PKG_FLAGS:=nonshared

ifeq ($(ARCH), x86)
	ifdef CONFIG_X86_32
		KLIBC_ARCH := i386
		asflags-y += -D_REGPARM
	else
		KLIBC_ARCH := x86_64
	endif
else ifeq ($(ARCH), aarch64)
	KLIBC_ARCH := arm64
else
	KLIBC_ARCH := $(ARCH)
endif

asflags-y += -I${PWD}$(KLIBC_USR)/include/arch/$(KLIBC_ARCH)
KLIBC_USR := /klibc/usr

EXTRA_KCONFIG:= \
	CONFIG_LUNATIK_RUN=m CONFIG_LUNATIK_RUNTIME=y CONFIG_LUNATIK_DEVICE=m   \
	CONFIG_LUNATIK_LINUX=m CONFIG_LUNATIK_NOTIFIER=m CONFIG_LUNATIK_SOCKET=m \
	CONFIG_LUNATIK_RCU=m CONFIG_LUNATIK_THREAD=m CONFIG_LUNATIK_FIB=m \
	CONFIG_LUNATIK_DATA=m CONFIG_LUNATIK_PROBE=m CONFIG_LUNATIK_SYSCALL=m \
	CONFIG_LUNATIK_XDP=m CONFIG_LUNATIK_FIFO=m CONFIG_LUNATIK_NETFILTER=m \
	CONFIG_LUNATIK_COMPLETION=m

EXTRA_CFLAGS:= -D_LUNATIK -D_KERNEL -DLUNATIK_RUNTIME=$(CONFIG_LUNATIK_RUNTIME) \
	-Wimplicit-fallthrough=0 -Wno-declaration-after-statement \
	-I$(PKG_BUILD_DIR) -I$(PKG_BUILD_DIR)/include -I$(PKG_BUILD_DIR)/lua \
	-I$(PKG_BUILD_DIR)$(KLIBC_USR)/include/arch/$(KLIBC_ARCH)
	#-I/home/marcel/development/marcelstanley/openwrt/build_dir/target-mips_24kc_musl/linux-ath79_nand/lunatik-3.5/klibc/usr/include/arch/mips

ASFLAGS += -I$(PKG_BUILD_DIR)$(KLIBC_USR)/include/arch/$(KLIBC_ARCH)
LUA_API:= lua/lua.h lua/lauxlib.h lua/lualib.h

LUNATIK_INSTALL_PATH:= /usr/sbin

SCRIPTS_INSTALL_PATH:= /lib/modules/lua

include $(INCLUDE_DIR)/package.mk

define Package/lunatik/Default
  TITLE:=Lunatik
  URL:=$PKG_SOURCE_URL
endef

define KernelPackage/lunatik
  $(call Package/lunatik/Default)
  SUBMENU:=Other modules
  KCONFIG:=CONFIG_NETFILTER=y CONFIG_NETFILTER_ADVANCED=y
  DEPENDS:=\
	   +kmod-nf-ipt \
	   +lunatik-scripts
  FILES:=$(PKG_BUILD_DIR)/lunatik.ko $(PKG_BUILD_DIR)/lunatik_run.ko \
	 $(PKG_BUILD_DIR)/lib/luarcu.ko $(PKG_BUILD_DIR)/lib/luadevice.ko \
	 $(PKG_BUILD_DIR)/lib/luadata.ko $(PKG_BUILD_DIR)/lib/luanetfilter.ko \
	 $(PKG_BUILD_DIR)/lib/luasocket.ko $(PKG_BUILD_DIR)/lib/luanotifier.ko \
	 $(PKG_BUILD_DIR)/lib/luathread.ko $(PKG_BUILD_DIR)/lib/lualinux.ko \
	 $(PKG_BUILD_DIR)/lib/luafifo.ko $(PKG_BUILD_DIR)/lib/luacompletion.ko
  AUTOLOAD:=$(call AutoLoad,30,lunatik,1)
endef

define KernelPackage/lunatik/description
  Lunatik is a framework for scripting the Linux kernel with Lua.
endef

define Package/lunatik-scripts
  $(call Package/lunatik/Default)
  DEPENDS:=\
	   +lua5.4
endef

define Package/lunatik-scripts/description
	Lunatik kernel scripts and userspace utilities
endef

define Package/lunatik-scripts/install
	$(INSTALL_DIR) $(1)$(LUNATIK_INSTALL_PATH)
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/bin/lunatik $(1)$(LUNATIK_INSTALL_PATH)
	$(INSTALL_DIR) $(1)$(SCRIPTS_INSTALL_PATH)
	$(INSTALL_DIR) $(1)$(SCRIPTS_INSTALL_PATH)/lunatik
	$(INSTALL_DIR) $(1)$(SCRIPTS_INSTALL_PATH)/socket
	$(INSTALL_DIR) $(1)$(SCRIPTS_INSTALL_PATH)/syscall
	$(INSTALL_DATA) $(PKG_BUILD_DIR)/driver.lua $(1)$(SCRIPTS_INSTALL_PATH)
	$(INSTALL_DATA) $(PKG_BUILD_DIR)/lib/mailbox.lua $(1)$(SCRIPTS_INSTALL_PATH)
	$(INSTALL_DATA) $(PKG_BUILD_DIR)/lib/lunatik/*.lua $(1)$(SCRIPTS_INSTALL_PATH)/lunatik
	$(INSTALL_DATA) $(PKG_BUILD_DIR)/lib/socket/*.lua $(1)$(SCRIPTS_INSTALL_PATH)/socket
	$(INSTALL_DATA) $(PKG_BUILD_DIR)/lib/syscall/*.lua $(1)$(SCRIPTS_INSTALL_PATH)/syscall
endef

define Build/Prepare
        $(Build/Prepare/Default)
	cd $(PKG_BUILD_DIR) && ./gensymbols.sh $(LUA_API) > lunatik_sym.h
	$(CP) ./Makefile.owrt $(PKG_BUILD_DIR)/Makefile
endef

define Build/Compile
	$(KERNEL_MAKE) M="$(PKG_BUILD_DIR)" EXTRA_CFLAGS="$(EXTRA_CFLAGS)" $(EXTRA_KCONFIG) TARGET_ASFLAGS="$(ASFLAGS)" modules
endef

$(eval $(call KernelPackage,lunatik))
$(eval $(call BuildPackage,lunatik-scripts))

include $(TOPDIR)/rules.mk

PKG_NAME:=qca-nss-drv
PKG_RELEASE:=$(AUTORELEASE)

PKG_SOURCE_URL:=https://git.codelinaro.org/clo/qsdk/oss/lklm/nss-drv.git
PKG_SOURCE_PROTO:=git
PKG_SOURCE_DATE:=2022-03-07
PKG_SOURCE_VERSION:=397c88cf184e0eb011dd44ad82c2dfae60ece1b4
PKG_MIRROR_HASH:=a3bea305d85fcec3d77f9b40d06a71b65ccf3d8b98018d9c8c23d6ad95e6aedc

PKG_BUILD_PARALLEL:=1

include $(INCLUDE_DIR)/kernel.mk
include $(INCLUDE_DIR)/package.mk

NSS_CLIENTS_DIR:=$(TOPDIR)/qca/src/qca-nss-clients

define KernelPackage/qca-nss-drv
  SECTION:=kernel
  CATEGORY:=Kernel modules
  SUBMENU:=Network Devices
  DEPENDS:=@(TARGET_ipq807x||TARGET_ipq60xx) +kmod-qca-nss-dp
  TITLE:=Kernel driver for NSS (core driver)
  FILES:=$(PKG_BUILD_DIR)/qca-nss-drv.ko
  AUTOLOAD:=$(call AutoLoad,32,qca-nss-drv)
endef

define KernelPackage/qca-nss-drv/install
	$(INSTALL_DIR) $(1)/lib/debug
	$(INSTALL_DIR) $(1)/etc/init.d
	$(INSTALL_DIR) $(1)/etc/sysctl.d
	$(INSTALL_DIR) $(1)/etc/hotplug.d/firmware
	$(INSTALL_DIR) $(1)/etc/config

	$(INSTALL_BIN) ./files/qca-nss-drv.debug $(1)/lib/debug/qca-nss-drv
	$(INSTALL_BIN) ./files/qca-nss-drv.init $(1)/etc/init.d/qca-nss-drv
	$(INSTALL_BIN) ./files/qca-nss-drv.sysctl $(1)/etc/sysctl.d/qca-nss-drv.conf
	$(INSTALL_BIN) ./files/qca-nss-drv.hotplug $(1)/etc/hotplug.d/firmware/10-qca-nss-fw
	$(INSTALL_BIN) ./files/qca-nss-drv.conf $(1)/etc/config/nss

endef

define KernelPackage/qca-nss-drv/Description
This package contains a NSS driver for QCA chipset
endef

define Build/InstallDev
	mkdir -p $(1)/usr/include/qca-nss-drv
	$(CP) $(PKG_BUILD_DIR)/exports/* $(1)/usr/include/qca-nss-drv/
endef

EXTRA_CFLAGS+= -I$(STAGING_DIR)/usr/include/qca-nss-gmac \
			   -I$(STAGING_DIR)/usr/include/qca-nss-dp \
			   -I$(STAGING_DIR)/usr/include/qca-ssdk \
			   -Wno-unused-variable

ifneq (, $(findstring $(CONFIG_TARGET_BOARD), "ipq807x" "ipq60xx"))
EXTRA_CFLAGS+= -DNSS_MEM_PROFILE_MEDIUM
LOW_MEM_PROFILE_MAKE_OPTS=y
endif

ifeq ($(CONFIG_KERNEL_SKB_FIXED_SIZE_2K),y)
EXTRA_CFLAGS+= -DNSS_SKB_FIXED_SIZE_2K
endif

DRV_MAKE_OPTS:=
ifeq ($(LOW_MEM_PROFILE_MAKE_OPTS),y)
DRV_MAKE_OPTS+=NSS_DRV_C2C_ENABLE=n \
	NSS_DRV_CAPWAP_ENABLE=n \
	NSS_DRV_CLMAP_ENABLE=n \
	NSS_DRV_CRYPTO_ENABLE=y \
	NSS_DRV_DTLS_ENABLE=n \
	NSS_DRV_GRE_ENABLE=n \
	NSS_DRV_GRE_REDIR_ENABLE=n \
	NSS_DRV_GRE_TUNNEL_ENABLE=n \
	NSS_DRV_IGS_ENABLE=n \
	NSS_DRV_IPSEC_ENABLE=n \
	NSS_DRV_LAG_ENABLE=n \
	NSS_DRV_L2TP_ENABLE=n \
	NSS_DRV_MAPT_ENABLE=n \
	NSS_DRV_OAM_ENABLE=n \
	NSS_DRV_PPTP_ENABLE=n \
	NSS_DRV_PORTID_ENABLE=n \
	NSS_DRV_PVXLAN_ENABLE=n \
	NSS_DRV_QRFS_ENABLE=n \
	NSS_DRV_QVPN_ENABLE=n \
	NSS_DRV_RMNET_ENABLE=n \
	NSS_DRV_SHAPER_ENABLE=n \
	NSS_DRV_SJACK_ENABLE=n \
	NSS_DRV_TLS_ENABLE=n \
	NSS_DRV_TRUSTSEC_ENABLE=n \
	NSS_DRV_TSTAMP_ENABLE=n \
	NSS_DRV_TUN6RD_ENABLE=n \
	NSS_DRV_TUNIPIP6_ENABLE=n \
	NSS_DRV_VXLAN_ENABLE=n \
	NSS_DRV_MATCH_ENABLE=n \
	NSS_DRV_MIRROR_ENABLE=n
endif

ifeq ($(CONFIG_TARGET_BOARD), "ipq807x")
    SOC="ipq807x_64"
else ifeq ($(CONFIG_TARGET_BOARD), "ipq60xx")
    SOC="ipq60xx_64"
endif

define Build/Configure
	$(LN) arch/nss_$(SOC).h $(PKG_BUILD_DIR)/exports/nss_arch.h
endef

define Build/Compile
	+$(MAKE) -C "$(LINUX_DIR)" $(strip $(DRV_MAKE_OPTS)) \
		CROSS_COMPILE="$(TARGET_CROSS)" \
		ARCH="$(LINUX_KARCH)" \
		M="$(PKG_BUILD_DIR)" \
		EXTRA_CFLAGS="$(EXTRA_CFLAGS)" SoC=$(SOC) \
		$(KERNEL_MAKE_FLAGS) \
		$(PKG_JOBS) \
		modules
endef

$(eval $(call KernelPackage,qca-nss-drv))

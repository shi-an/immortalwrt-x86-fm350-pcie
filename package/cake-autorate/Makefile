include $(TOPDIR)/rules.mk

PKG_NAME:=cake-autorate
PKG_VERSION:=3.2.2
PKG_RELEASE:=1

PKG_SOURCE:=$(PKG_NAME)-$(PKG_VERSION).tar.gz
PKG_SOURCE_URL:=https://codeload.github.com/lynxthecat/cake-autorate/tar.gz
PKG_SOURCE_URL_FILE:=v$(PKG_VERSION)
PKG_HASH:=892d8e648f6b3705f31799736e697874da3802b5e56ce4aea257cfdf6a376414

PKG_LICENSE:=GPL-2.0-only
PKG_LICENSE_FILES:=LICENCE.md

include $(INCLUDE_DIR)/package.mk

define Package/cake-autorate
  SECTION:=net
  CATEGORY:=Network
  TITLE:=Adaptive bandwidth control for CAKE
  URL:=https://github.com/lynxthecat/cake-autorate
  DEPENDS:=+bash +fping +tc +sqm-scripts
  PKGARCH:=all
endef

define Package/cake-autorate/description
 Adjust existing CAKE shaper rates according to traffic load and latency.
 Configure SQM and /etc/cake-autorate/config.primary.sh before enabling
 the service through /etc/config/cake-autorate.
endef

define Package/cake-autorate/conffiles
/etc/config/cake-autorate
/etc/cake-autorate/config.primary.sh
endef

define Build/Configure
endef

define Build/Compile
endef

define Package/cake-autorate/install
	$(INSTALL_DIR) $(1)/usr/lib/cake-autorate
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/cake-autorate.sh $(1)/usr/lib/cake-autorate/
	$(INSTALL_DATA) $(PKG_BUILD_DIR)/defaults.sh $(PKG_BUILD_DIR)/lib.sh $(1)/usr/lib/cake-autorate/
	$(INSTALL_DIR) $(1)/etc/cake-autorate $(1)/etc/config $(1)/etc/init.d
	$(INSTALL_CONF) $(PKG_BUILD_DIR)/config.primary.sh $(1)/etc/cake-autorate/
	$(INSTALL_CONF) ./files/cake-autorate.config $(1)/etc/config/cake-autorate
	$(INSTALL_BIN) ./files/cake-autorate.init $(1)/etc/init.d/cake-autorate
	$(INSTALL_DIR) $(1)/lib/upgrade/keep.d
	$(INSTALL_DATA) ./files/cake-autorate.keep $(1)/lib/upgrade/keep.d/cake-autorate
	$(INSTALL_DIR) $(1)/usr/share/doc/cake-autorate
	$(INSTALL_DATA) $(PKG_BUILD_DIR)/LICENCE.md $(1)/usr/share/doc/cake-autorate/
endef

$(eval $(call BuildPackage,cake-autorate))

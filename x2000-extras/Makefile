include $(TOPDIR)/rules.mk

PKG_NAME:=x2000-extras
PKG_VERSION:=1
PKG_RELEASE:=2
PKG_LICENSE:=GPL-2.0-only
PKG_MAINTAINER:=Gilberto Conde <gilbertorconde@gmail.com>

include $(INCLUDE_DIR)/package.mk

define Package/x2000-extras
  SECTION:=utils
  CATEGORY:=Utilities
  TITLE:=GL.iNet GL-X2000 extras: LEDs, mode button, cellular + WAN failover
  DEPENDS:=+modemmanager +kmod-usb-net-qmi-wwan +kmod-usb-serial-option \
	+mwan3
  PKGARCH:=all
endef

define Package/x2000-extras/description
  Fills in what the OEM firmware did on its own on the GL-X2000:
  - Internet LED from real connectivity (mwan3-aware), signal bars from
    ModemManager
  - reset button: hold 2-5s to toggle home (Wi-Fi off, backup WAN) and
    travel (Wi-Fi on) mode; factory reset moved to 10s
  - modem + ethernet WAN uplinks with mwan3 failover (ethernet first)
  - watchdog that brings the modem back after a carrier hangup
endef

define Package/x2000-extras/conffiles
/etc/router-mode
/etc/router-mode.radios
endef

define Build/Compile
endef

define Package/x2000-extras/install
	$(INSTALL_DIR) $(1)/usr/sbin
	$(INSTALL_BIN) ./files/router-mode $(1)/usr/sbin/
	$(INSTALL_BIN) ./files/internet-led $(1)/usr/sbin/
	$(INSTALL_BIN) ./files/cellular-leds $(1)/usr/sbin/
	$(INSTALL_BIN) ./files/wan-watchdog $(1)/usr/sbin/
	$(INSTALL_DIR) $(1)/usr/libexec/x2000-extras
	$(INSTALL_BIN) ./files/reset-button $(1)/usr/libexec/x2000-extras/
	$(INSTALL_DIR) $(1)/etc/init.d
	$(INSTALL_BIN) ./files/x2000-leds.init $(1)/etc/init.d/x2000-leds
	$(INSTALL_DIR) $(1)/etc/hotplug.d/iface
	$(INSTALL_DATA) ./files/x2000-leds.hotplug $(1)/etc/hotplug.d/iface/99-x2000-leds
	$(INSTALL_DIR) $(1)/etc/uci-defaults
	$(INSTALL_BIN) ./files/x2000-extras.defaults $(1)/etc/uci-defaults/90-x2000-extras
endef

# On a running box, apply what the uci-defaults just changed. In an image the
# uci-defaults run at first boot, before these services start.
define Package/x2000-extras/postinst
#!/bin/sh
[ -n "$${IPKG_INSTROOT}" ] && exit 0
/etc/init.d/led reload
wifi reload
/etc/init.d/network reload
/etc/init.d/firewall reload
/etc/init.d/mwan3 restart
/etc/init.d/cron restart
exit 0
endef

define Package/x2000-extras/prerm
#!/bin/sh
[ -n "$${IPKG_INSTROOT}" ] && exit 0
[ -e /etc/rc.button/reset.stock ] && mv /etc/rc.button/reset.stock /etc/rc.button/reset
sed -i '\#/usr/sbin/wan-watchdog#d' /etc/crontabs/root
/etc/init.d/cron restart
exit 0
endef

$(eval $(call BuildPackage,x2000-extras))

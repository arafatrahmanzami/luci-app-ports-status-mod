include $(TOPDIR)/rules.mk

PKG_NAME:=luci-app-ports-status-mod
PKG_VERSION:=1.0.14
PKG_RELEASE:=1

LUCI_TITLE:=LuCI app for enhanced port status monitoring
LUCI_DESCRIPTION:=Modified port status with labels, descriptions, drag-drop ordering, port enable/disable, and status dots.
LUCI_PKGARCH:=all
LUCI_DEPENDS:=+luci-base +luci-mod-status +rpcd-mod-file

PKG_MAINTAINER:=Arafat Rahman Zami
PKG_LICENSE:=MIT
PKG_LICENSE_FILES:=LICENSE

include $(TOPDIR)/feeds/luci/luci.mk

# call BuildPackage - OpenWrt buildroot signature

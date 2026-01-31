include $(TOPDIR)/rules.mk

PKG_NAME:=nfqws2
PKG_RELEASE:=1

include $(INCLUDE_DIR)/package.mk
include $(INCLUDE_DIR)/host-build.mk

LUA_JIT?=1

ifeq ($(LUA_JIT),1)
	LUA_VER?=5.1
	LUA_DEPEND:=
	LUAJIT_VER?=2.1
	ifeq ($(ARCH),riscv64)
		LUAJIT_URL:=https://github.com/infiWang/luajit2/archive
		LAUJIT_BRANCH:=v2.1-20260114-riscv64
		LUAJIT_COMMIT=4d8cf882a3a4ed3259407ad78614a60b02dd553f
		LUAJIT_RELEASE:=2.1-20260114-$(LUAJIT_COMMIT)
		LUAJIT_URL_FILE:=$(LUAJIT_COMMIT).tar.gz
	else
		LUAJIT_URL:=https://github.com/openresty/luajit2/archive/refs/tags
		LUAJIT_RELEASE:=2.1-20250826
		LUAJIT_URL_FILE:=v$(LUAJIT_RELEASE).tar.gz
	endif
	LUAJIT_TGZ:=$(PKG_NAME)-luajit-$(LUAJIT_RELEASE).tar.gz
	LUASRC_DIR:=$(PKG_BUILD_DIR)/luajit-$(LUAJIT_RELEASE)
	LUAOUT_DIR:=$(PKG_BUILD_DIR)/luajit-out
	LUA_INCLUDE:=-I$(LUAOUT_DIR)/include/luajit-$(LUAJIT_VER)
	LUA_LIBRARY:=-L$(LUAOUT_DIR)/lib -lluajit-$(LUA_VER)
else
	LUA_VER?=5.4
	LUA_DEPEND:= +lua$(LUA_VER)
	LUA_INCLUDE:=-I$(STAGING_DIR)/usr/include/lua$(LUA_VER)
	LUA_LIBRARY:=-L$(STAGING_DIR)/usr/lib -llua$(LUA_VER)
endif

define Package/nfqws2
	SECTION:=net
	CATEGORY:=Network
	TITLE:=$(PKG_NAME)
	SUBMENU:=Zapret2
	DEPENDS:=+libnetfilter-queue +libcap +zlib +$(LUA_DEP)
endef

ifeq ($(LUA_JIT),1)
define Download/$(PKG_NAME)-luajit
	FILE:=$(LUAJIT_TGZ)
	URL:=$(LUAJIT_URL)
	URL_FILE:=$(LUAJIT_URL_FILE)
	HASH:=skip
endef
$(eval $(call Download,$(PKG_NAME)-luajit))
endif

define Build/Prepare
	$(Build/Prepare/Default)
	mkdir -p $(PKG_BUILD_DIR)
	$(CP) ./nfq2/* $(PKG_BUILD_DIR)/
	if [ "$(LUA_JIT)" = 1 ]; then \
		if [ ! -f "$(DL_DIR)/$(LUAJIT_TGZ)" ]; then \
			echo "ERROR: LuaJIT tarball not found:"; \
			echo "  $(LUAJIT_TGZ)"; \
			echo "Run: make package/$(PKG_NAME)/download"; \
			false; \
		fi ; \
		rm -rf $(LUASRC_DIR) ; \
		mkdir -p $(LUASRC_DIR) ; \
		tar -xzf "$(DL_DIR)/$(LUAJIT_TGZ)" --strip-components=1 -C "$(LUASRC_DIR)" ; \
		rm -rf $(LUAOUT_DIR) ; \
	fi
endef

ifeq ($(HOST_ARCH),$(filter $(HOST_ARCH), x86_64 mips64))
  ifeq ($(CONFIG_ARCH_64BIT),)
    HOST_BITS := -m32
  endif
endif

define Build/Compile
	if [ "$(LUA_JIT)" = 1 ]; then \
		$(MAKE) $(PKG_JOBS) -C $(LUASRC_DIR) \
			HOST_CC="$(HOSTCC) $(HOST_CFLAGS) $(HOST_BITS)" \
			CROSS="$(TARGET_CROSS)" \
			XCFLAGS="-DLUAJIT_DISABLE_FFI" \
			TARGET_CFLAGS="-Oz -flto=auto -ffunction-sections -fdata-sections $(TARGET_CFLAGS)" \
			TARGET_LDFLAGS="-flto=auto $(TARGET_LDFLAGS)" \
			TARGET_AR="$(TARGET_AR) rcus" \
			BUILDMODE=static \
			DESTDIR=$(LUAOUT_DIR) \
			PREFIX= \
			TARGET_SYS=Linux ; \
		rm -rf $(LUAOUT_DIR) ; \
		if [ ! -f $(LUASRC_DIR)/src/libluajit.a ]; then \
			echo "ERROR: file libluajit.a not found!"; \
			false; \
		fi ; \
		mkdir -p $(LUAOUT_DIR) ; \
		$(MAKE) -C $(LUASRC_DIR) \
			DESTDIR=$(LUAOUT_DIR) \
			PREFIX= \
			TARGET_SYS=Linux \
			install ; \
	fi
	$(MAKE) -C $(PKG_BUILD_DIR)/$(MAKE_PATH) $(MAKE_FLAGS) LUA_JIT=$(LUA_JIT) LUA_CFLAGS="$(LUA_INCLUDE)" LUA_LIB="$(LUA_LIBRARY)"
endef

define Package/$(PKG_NAME)/install
	$(INSTALL_DIR) $(1)/opt/zapret2/binaries/my
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/nfqws2 $(1)/opt/zapret2/binaries/my
endef

$(eval $(call BuildPackage,$(PKG_NAME)))

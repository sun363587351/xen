XEN_ROOT = $(CURDIR)/../../..
include $(XEN_ROOT)/tools/Rules.mk

MAJOR    = 1
MINOR    = 1
SHLIB_LDFLAGS += -Wl,--version-script=libxenforeignmemory.map

CFLAGS   += -Werror -Wmissing-prototypes
CFLAGS   += -I./include $(CFLAGS_xeninclude)
CFLAGS   += $(CFLAGS_libxentoollog)

SRCS-y                 += core.c
SRCS-$(CONFIG_Linux)   += linux.c
SRCS-$(CONFIG_FreeBSD) += freebsd.c
SRCS-$(CONFIG_SunOS)   += compat.c solaris.c
SRCS-$(CONFIG_NetBSD)  += compat.c netbsd.c
SRCS-$(CONFIG_MiniOS)  += minios.c

LIB_OBJS := $(patsubst %.c,%.o,$(SRCS-y))
PIC_OBJS := $(patsubst %.c,%.opic,$(SRCS-y))

LIB := libxenforeignmemory.a
ifneq ($(nosharedlibs),y)
LIB += libxenforeignmemory.so
endif

PKG_CONFIG := xenforeignmemory.pc
PKG_CONFIG_VERSION := $(MAJOR).$(MINOR)

ifneq ($(CONFIG_LIBXC_MINIOS),y)
PKG_CONFIG_INST := $(PKG_CONFIG)
$(PKG_CONFIG_INST): PKG_CONFIG_PREFIX = $(prefix)
$(PKG_CONFIG_INST): PKG_CONFIG_INCDIR = $(includedir)
$(PKG_CONFIG_INST): PKG_CONFIG_LIBDIR = $(libdir)
endif

PKG_CONFIG_LOCAL := $(foreach pc,$(PKG_CONFIG),$(PKG_CONFIG_DIR)/$(pc))

$(PKG_CONFIG_LOCAL): PKG_CONFIG_PREFIX = $(XEN_ROOT)
$(PKG_CONFIG_LOCAL): PKG_CONFIG_INCDIR = $(XEN_LIBXENFOREIGNMEMORY)/include
$(PKG_CONFIG_LOCAL): PKG_CONFIG_LIBDIR = $(CURDIR)
$(PKG_CONFIG_LOCAL): PKG_CONFIG_CFLAGS_LOCAL = $(CFLAGS_xeninclude)

.PHONY: all
all: build

.PHONY: build
build:
	$(MAKE) libs

.PHONY: libs
libs: headers.chk $(LIB) $(PKG_CONFIG_INST) $(PKG_CONFIG_LOCAL)

headers.chk: $(wildcard include/*.h)

libxenforeignmemory.a: $(LIB_OBJS)
	$(AR) rc $@ $^

libxenforeignmemory.so: libxenforeignmemory.so.$(MAJOR)
	$(SYMLINK_SHLIB) $< $@
libxenforeignmemory.so.$(MAJOR): libxenforeignmemory.so.$(MAJOR).$(MINOR)
	$(SYMLINK_SHLIB) $< $@

libxenforeignmemory.so.$(MAJOR).$(MINOR): $(PIC_OBJS) libxenforeignmemory.map
	$(CC) $(LDFLAGS) $(PTHREAD_LDFLAGS) -Wl,$(SONAME_LDFLAG) -Wl,libxenforeignmemory.so.$(MAJOR) $(SHLIB_LDFLAGS) -o $@ $(PIC_OBJS) $(LDLIBS_libxentoollog) $(APPEND_LDFLAGS)

.PHONY: install
install: build
	$(INSTALL_DIR) $(DESTDIR)$(libdir)
	$(INSTALL_DIR) $(DESTDIR)$(includedir)
	$(INSTALL_SHLIB) libxenforeignmemory.so.$(MAJOR).$(MINOR) $(DESTDIR)$(libdir)
	$(INSTALL_DATA) libxenforeignmemory.a $(DESTDIR)$(libdir)
	$(SYMLINK_SHLIB) libxenforeignmemory.so.$(MAJOR).$(MINOR) $(DESTDIR)$(libdir)/libxenforeignmemory.so.$(MAJOR)
	$(SYMLINK_SHLIB) libxenforeignmemory.so.$(MAJOR) $(DESTDIR)$(libdir)/libxenforeignmemory.so
	$(INSTALL_DATA) include/xenforeignmemory.h $(DESTDIR)$(includedir)
	$(INSTALL_DATA) xenforeignmemory.pc $(DESTDIR)$(PKG_INSTALLDIR)

.PHONY: TAGS
TAGS:
	etags -t *.c *.h

.PHONY: clean
clean:
	rm -rf *.rpm $(LIB) *~ $(DEPS) $(LIB_OBJS) $(PIC_OBJS)
	rm -f libxenforeignmemory.so.$(MAJOR).$(MINOR) libxenforeignmemory.so.$(MAJOR)
	rm -f headers.chk
	rm -f xenforeignmemory.pc

.PHONY: distclean
distclean: clean

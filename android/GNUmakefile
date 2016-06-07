override MYDIR := $(abspath $(dir $(lastword $(MAKEFILE_LIST))))
override TOPDIR := $(abspath $(MYDIR)/..)

sinclude $(TOPDIR)/local.mk

#ABI ?= armeabi-v7a-hard
ABI ?= arm64-v8a

ifeq (,$(strip $(NDK)))
NDK := $(abspath $(wildcard $(TOPDIR)/../../platform/ndk))
endif

#==================================================================================================

define rm-if-exists
$(if $(wildcard $(1)),rm -Rf $(wildcard $(1)))
endef

define out-dir
$(or $(strip $(OUTDIR)),$(TOPDIR)/android/out/$(1))
endef

define build-dir
$(call out-dir,$(1))/build
endef

define install-dir
$(call out-dir,$(1))/install
endef

#==================================================================================================

.PHONY: all
all: build

.PHONY: clean
clean:
	$(call rm-if-exists,$(call out-dir))

.PHONY: rebuild
rebuild:
	$(MAKE) clean
	$(MAKE) all

.PHONY: build
build:
	$(if $(strip $(NDK)),,$(error NDK was not defined!))
	$(if $(strip $(ABI)),,$(error ABI was not defined!))
	$(if $(strip $(NCURSES)),,$(error NCURSES was not defined!))
	@$(strip \
		export NDK=$(NDK) && \
		export ABI=$(ABI) && \
		export NCURSES=$(NCURSES) && \
		export OUTDIR=$(call out-dir,$(ABI)) && \
		source $(TOPDIR)/android/envsetup.sh && \
		rm -Rf $(call build-dir,$(ABI)) && \
		mkdir -p $(call build-dir,$(ABI)) && \
		rsync -a --delete --exclude=.git --exclude=android/ $(TOPDIR)/ $(call build-dir,$(ABI))/ && \
		cd $(call build-dir,$(ABI)) && \
		export LIBS='-ltinfow' && \
		vim_cv_getcwd_broken=no \
		vim_cv_memmove_handles_overlap=yes \
		vim_cv_stat_ignores_slash=yes \
		vim_cv_terminfo=yes \
		vim_cv_toupper_broken=no \
		vim_cv_tty_group=world \
		./configure --prefix=$(call install-dir,$(ABI)) \
			--host=$$TARGET \
			--disable-nls \
			--with-tlib=ncurses \
			&& \
		$(MAKE) -j && \
		$(MAKE) install && \
		true \
	)

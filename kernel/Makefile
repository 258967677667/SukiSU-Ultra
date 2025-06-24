kernelsu-objs := ksu.o
kernelsu-objs += allowlist.o
kernelsu-objs += apk_sign.o
kernelsu-objs += sucompat.o
kernelsu-objs += throne_tracker.o
kernelsu-objs += core_hook.o
kernelsu-objs += ksud.o
kernelsu-objs += embed_ksud.o
kernelsu-objs += kernel_compat.o

kernelsu-objs += selinux/selinux.o
kernelsu-objs += selinux/sepolicy.o
kernelsu-objs += selinux/rules.o
ccflags-y += -I$(srctree)/security/selinux -I$(srctree)/security/selinux/include
ccflags-y += -I$(objtree)/security/selinux -include $(srctree)/include/uapi/asm-generic/errno.h

obj-$(CONFIG_KSU) += kernelsu.o

obj-$(CONFIG_KPM) += kpm/


REPO_OWNER := SukiSU-Ultra
REPO_NAME := SukiSU-Ultra
KSU_API_VERSION_LOCAL := 3.1.6

GIT_BIN := /usr/bin/env PATH="$$PATH":/usr/bin:/usr/local/bin git
CURL_BIN := /usr/bin/env PATH="$$PATH":/usr/bin:/usr/local/bin curl

KSU_GITHUB_VERSION := $(shell $(CURL_BIN) -s "https://api.github.com/repos/$(REPO_OWNER)/$(REPO_NAME)/releases/latest" | grep '"tag_name":' | sed -E 's/.*"v([^"]+)".*/\1/')

LOCAL_GIT_EXISTS := $(shell test -e $(srctree)/$(src)/../.git && echo 1 || echo 0)

define get_ksu_api_version
$(shell echo $1 | awk -F'[-.]' '{printf "%d%02d%02d", $$1, $$2, $$3}')
endef

define get_ksu_kernel_version
v$1-$(shell cd $(srctree)/$(src); $(GIT_BIN) rev-parse --short=8 HEAD)@$(shell cd $(srctree)/$(src); $(GIT_BIN) rev-parse --abbrev-ref HEAD)
endef

ifeq ($(KSU_GITHUB_VERSION),)
    ifeq ($(LOCAL_GIT_EXISTS),1)
        $(shell cd $(srctree)/$(src); [ -f ../.git/shallow ] && $(GIT_BIN) fetch --unshallow)
        KSU_API_VERSION := $(call get_ksu_api_version,$(KSU_API_VERSION_LOCAL))
        KSU_KERNEL_VERSION := $(call get_ksu_kernel_version,$(KSU_API_VERSION_LOCAL))
        $(info -- $(REPO_NAME) version (local .git): $(KSU_KERNEL_VERSION))
        $(info -- $(REPO_NAME) Formatted version (local .git): $(KSU_API_VERSION))
    else
        KSU_KERNEL_VERSION := v$(KSU_API_VERSION_LOCAL)-$(REPO_NAME)-unknown@unknown
        KSU_API_VERSION := $(call get_ksu_api_version,$(KSU_API_VERSION_LOCAL))
        $(warning -- $(REPO_NAME) version: $(KSU_KERNEL_VERSION))
    endif
else
    $(shell cd $(srctree)/$(src); [ -f ../.git/shallow ] && $(GIT_BIN) fetch --unshallow)
    KSU_API_VERSION := $(call get_ksu_api_version,$(KSU_GITHUB_VERSION))
    KSU_KERNEL_VERSION := $(call get_ksu_kernel_version,$(KSU_GITHUB_VERSION))
    $(info -- $(REPO_NAME) version (Github): $(KSU_KERNEL_VERSION))
    $(info -- $(REPO_NAME) Formatted version (Github): $(KSU_API_VERSION))
endif

ccflags-y += -DKSU_VERSION=\"$(KSU_KERNEL_VERSION)\"
ccflags-y += -DKSU_API_VERSION=$(KSU_API_VERSION)

ifndef KSU_EXPECTED_SIZE
KSU_EXPECTED_SIZE := 0x35c
endif

ifndef KSU_EXPECTED_HASH
KSU_EXPECTED_HASH := 947ae944f3de4ed4c21a7e4f7953ecf351bfa2b36239da37a34111ad29993eef
endif

ifdef KSU_MANAGER_PACKAGE
ccflags-y += -DKSU_MANAGER_PACKAGE=\"$(KSU_MANAGER_PACKAGE)\"
$(info -- SukiSU Manager package name: $(KSU_MANAGER_PACKAGE))
endif

$(info -- SukiSU Manager signature size: $(KSU_EXPECTED_SIZE))
$(info -- SukiSU Manager signature hash: $(KSU_EXPECTED_HASH))
$(info -- Supported Unofficial Manager: 5ec1cff (GKI) ShirkNeko udochina (GKI and KPM))
KERNEL_VERSION := $(VERSION).$(PATCHLEVEL)
KERNEL_TYPE := Non-GKI
# Check for GKI 2.0 (5.10+ or 6.x+)
ifneq ($(shell test \( $(VERSION) -ge 5 -a $(PATCHLEVEL) -ge 10 \) -o $(VERSION) -ge 6; echo $$?),0)
# Check for GKI 1.0 (5.4)
ifeq ($(shell test $(VERSION)-$(PATCHLEVEL) = 5-4; echo $$?),0)
KERNEL_TYPE := GKI 1.0
endif
else
KERNEL_TYPE := GKI 2.0
endif
$(info -- KERNEL_VERSION: $(KERNEL_VERSION))
$(info -- KERNEL_TYPE: $(KERNEL_TYPE))

$(info -- KERNEL_VERSION: $(KERNEL_VERSION))
ifeq ($(CONFIG_KPM),y)
$(info -- KPM is enabled)
else
$(info -- KPM is disabled)
endif


ccflags-y += -DEXPECTED_SIZE=$(KSU_EXPECTED_SIZE)
ccflags-y += -DEXPECTED_HASH=\"$(KSU_EXPECTED_HASH)\"

ccflags-y += -Wno-implicit-function-declaration -Wno-strict-prototypes -Wno-int-conversion -Wno-gcc-compat
ccflags-y += -Wno-declaration-after-statement -Wno-unused-function

# Keep a new line here!! Because someone may append config

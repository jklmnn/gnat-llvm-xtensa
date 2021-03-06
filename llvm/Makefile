# Note that this Makefile is optional and mainly used by core GNAT LLVM
# developers: you can use an existing install of LLVM instead.
# llvm-interface/Makefile will work as long as llvm-config is found in your
# PATH.

PWD:=$(shell pwd)
TMP_GEN_DIR=gen
OUT_DIR=out
#LLVM_RELEASE=9.0.1
LLVM_SRC_DIR=$(LLVM_GIT_DIR)/llvm
LLVM_INCLUDE_DIR=$(PWD)/$(LLVM_SRC_DIR)/include
LLVM_INCLUDE_DIR_C=$(LLVM_INCLUDE_DIR)/llvm-c
LLVM_GIT_DIR=llvm-xtensa

.PHONY: configure

all: setup llvm

binding: $(OUT_DIR)/llvm-core.adb

CONFIGURE=cd llvm-obj && CXX=g++ CC=gcc CXXC=g++ \
  cmake -G "Unix Makefiles" -DCMAKE_INSTALL_PREFIX=$(PWD)/install \
  -DCMAKE_EXE_LINKER_FLAGS="-static-libgcc -static-libstdc++" \
  -DLLVM_TARGETS_TO_BUILD="X86" \
  -DLLVM_ENABLE_PROJECTS="clang" \
  -DLLVM_EXPERIMENTAL_TARGETS_TO_BUILD="Xtensa" ../$(LLVM_SRC_DIR)

setup: $(LLVM_SRC_DIR) llvm-obj

llvm-obj:
	mkdir -p llvm-obj
	$(CONFIGURE)

configure:
	mkdir -p llvm-obj
	$(CONFIGURE)

configure-no-debug:
	mkdir -p llvm-obj
	$(CONFIGURE) -DCMAKE_BUILD_TYPE=Release -DLLVM_ENABLE_ASSERTIONS=ON

configure-prod:
	mkdir -p llvm-obj
	$(CONFIGURE) -DCMAKE_BUILD_TYPE=RelWithDebInfo

llvm:
	$(MAKE) -s -C llvm-obj

$(OUT_DIR)/llvm-core.adb: $(LLVM_INCLUDE_DIR_C)/Core.h py/common.py py/lxml_subp.py py/wrapper.py
	mkdir -p $(TMP_GEN_DIR) $(OUT_DIR)
	cd $(TMP_GEN_DIR) && g++ -I$(LLVM_INCLUDE_DIR) -I$(PWD)/llvm-obj/include -DNDEBUG -D_GNU_SOURCE -D__STDC_CONSTANT_MACROS -D__STDC_FORMAT_MACROS -D__STDC_LIMIT_MACROS -U__cplusplus -c -D_Bool=bool -Dlto_api_version=lto_api_version_fun -fdump-ada-spec -C $(LLVM_INCLUDE_DIR_C)/*.h
	sed -e '/Ada_2005/d' -i $(TMP_GEN_DIR)/*.ads
	sed -e 's/pragma Import (CPP, \(.*\), ".*");/pragma Import (C, \1, "\1");/g' -e "s^$(PWD)/^^g" -i $(TMP_GEN_DIR)/llvm_c_*_h.ads
	sed -e 's/\(with Interfaces.C; use Interfaces.C;\)/pragma Warnings (Off); \1 pragma Warnings (On);/g' -i $(TMP_GEN_DIR)/llvm_c_*_h.ads
	cd $(TMP_GEN_DIR) && gnat2xml -q -mxml llvm_c_*_h.ads
	echo -n "pragma Style_Checks (Off); package LLVM is end LLVM;" > $(OUT_DIR)/llvm.ads
	./py/common.py process_names gen/llvm_c_*_h.ads
	cd $(OUT_DIR) && rm x86_64_linux*.ads
	sed -e 's/x86_64_linux_gnu_bits_stdint_u*intn_h/stdint_h/g' -i $(OUT_DIR)/llvm-*.ads
	sed -e 's/x86_64_linux_gnu_sys_types_h/stddef_h/g' -i $(OUT_DIR)/llvm-*.ads
	./py/undupwiths.py out/llvm*.ads
	cp py/stddef_h.ads.proto $(OUT_DIR)/stddef_h.ads
	cp py/stdint_h.ads.proto $(OUT_DIR)/stdint_h.ads
	cd $(OUT_DIR) && gnat2xml -q -mxml llvm*.ads
	rm -rf gen
	mv out gen
	mkdir out
	./py/common.py generate_wrappers gen/llvm*.ads
	ls gen/*.ads | grep -v llvm | xargs -i cp {} out/
	rm -rf gen

clean:
	rm -rf $(TMP_GEN_DIR) $(OUT_DIR) llvm-obj

distclean: clean
	rm -rf $(LLVM_SRC_DIR)


# riscv_build_process
This repositiotory only for 
- riscv gnu tool-chain
- spike(simulator)
- pk(proxy kernel) 

build process

# source
## riscv-gnu-toolchain
https://github.com/riscv-collab/riscv-gnu-toolchain --branch 2023.06.02
this repository need submodule,see readme.md

## spike
git clone https://github.com/riscv-software-src/riscv-isa-sim --branch dummy-tag-for-ci-storage

## pk
git clone https://github.com/riscv-software-src/riscv-pk --branch v1.0.0

build process follow the readme.md, but the configure command need add --with-arch=rv64gc_zifencei


### issue https://github.com/riscv-software-src/riscv-pk/issues/260
Try configuring with --with-arch=rv64gc_zifencei (or whatever is appropriate for your target). More recent assemblers require the zifencei extension to be explicitly listed for the fence.i instruction to be available.




# HOW TO ENABLE RVV?

## build development environment for riscv-v vector extension
### riscv-gnu-toolchain
``` bash
git clone https://github.com/riscv/riscv-gnu-toolchain
cd riscv-gnu-toolchain
./configure --prefix=/home/cluster/code/env/riscv-gnu-toolchain-bin
make -j32 
make install
``` 

### qumu
``` bash
git clone https://github.com/qemu/qemu
cd qemu
mkdir build
cd build
../configure --target-list=riscv64-softmmu,riscv64-linux-user  --prefix=/home/cluster/code/env/qemu-riscv
make -j32
``` 


### llvm-project
``` bash
git clone https://github.com/llvm/llvm-project.git
cd llvm-project
mkdir build 
cd build 
cmake -G "Unix Makefiles" -DLLVM_TARGETS_TO_BUILD="RISCV"     -DLLVM_DEFAULT_TARGET_TRIPLE=riscv64-unknown-elf -DCMAKE_BUILD_TYPE=Release -DLLVM_ENABLE_PROJECTS="clang;lld" ../llvm
make -j32 
make install 
``` 

## run process

### srvv_strlen.c example
``` c
#include "common.h"
#include <riscv_vector.h>
#include <string.h>
// reference https://github.com/riscv/riscv-v-spec/blob/master/example/strlen.s
size_t strlen_vec(char *src) {
  size_t vlmax = __riscv_vsetvlmax_e8m8();
  char *copy_src = src;
  long first_set_bit = -1;
  size_t vl;
  while (first_set_bit < 0) {
    vint8m8_t vec_src = __riscv_vle8ff_v_i8m8(copy_src, &vl, vlmax);
    vbool1_t string_terminate = __riscv_vmseq_vx_i8m8_b1(vec_src, 0, vl);

    copy_src += vl;

    first_set_bit = __riscv_vfirst_m_b1(string_terminate, vl);
  }
  copy_src -= vl - first_set_bit;

  return (size_t)(copy_src - src);
}

int main() {
  const uint32_t seed = 0xdeadbeef;
  srand(seed);

  int N = rand() % 2000;

  // data gen
  char s0[N];
  gen_string(s0, N);

  // compute
  size_t golden, actual;
  golden = strlen(s0);
  actual = strlen_vec(s0);

  // compare
  puts(golden == actual ? "pass" : "fail");
}
``` 
### makefile(compile)
``` makefile
func := cltz 
target := a.out
source := rvv_strlen.c

MARCH := rv64gv
# MARCH := rv64gcv1p0_zfh_zvfh0p1 
CLANG := /home/cluster/code/env/llvm-project/build/bin/clang
RISCV := /home/cluster/code/env/riscv-gnu-toolchain-bin

compile:
	${CLANG} \
	--target=riscv64-unknown-elf \
	--sysroot=$(RISCV)/riscv64-unknown-elf \
	-march=$(MARCH) \
	-O2 \
	-menable-experimental-extensions \
	-mllvm \
	--riscv-v-vector-bits-min=256 \
	--gcc-toolchain=$(RISCV) \
	$(source) -o $(target)
```

### run (use qemu simulator)
``` bash
cd qeum/build/bin
./qemu-riscv64 -cpu rv64,v=true,vlen=256,vext_spec=v1.0 -L ~/code/env/riscv-gnu-toolchain/ ~/code/rvv-intrinsic-doc/examples/a.out 
```

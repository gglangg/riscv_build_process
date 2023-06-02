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

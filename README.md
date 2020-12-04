# Integrate-NVDLA-and-TVM

Official code for NVDLA software [sw](https://github.com/nvdla/sw). Official document [sw doc](http://nvdla.org/sw/contents.html). # Dess
6. For using nvdla_runtime to generate inference output, this will take a long time. If yo run this in a docker container, use `docker commit` to save your container, or after you exit and restart it, all message will lost.

## Current Progress:
1. Succesfully run Lenet written in relay on NVDLA hardware simulator.


## <span id="rebuild"> How to change and rebuild compiler</span> 
NVDLA Compiler can be updated using source code and rebuild as below. Ref: [modifying-nvdla-compiler](https://github.com/prasshantg/personal#modifying-nvdla-compiler)
```
cd {sw-repo-root}/umd
export TOP={sw-repo-root}/umd
make compiler
```
The rebuilt compiler is in `./out/apps/compiler/nvdla_compiler`, copy libnvdla_compiler.so to the same folder to use the rebuilt compiler:
`cp <path to sw>/sw/umd/out/core/src/compiler/libnvdla_compiler/libnvdla_compiler.so <path to sw>/sw/umd/out/apps/compiler/nvdla_compiler/`

In some cases if compiler build fails because of linking error with protobuf library then rebuild protobuf library as below 
```
cd <path to sw>/sw/umd/external/protobuf-2.6
./configure --enable-shared
make
make check
sudo make install
```
## How to Run the Whole Process for Model Inference

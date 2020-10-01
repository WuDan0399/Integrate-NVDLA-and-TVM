# Integrate-NVDLA-and-TVM(Developing)

## Some notes 
1. Official code for NVDLA software [sw](https://github.com/nvdla/sw). Official document [sw doc](http://nvdla.org/sw/contents.html). But they currently go down (Sep 30 2020). Home page reopen on Oct 1 2020, but links are still under maintanance.
2.  To compile the umd part in sw, use command `make <path to sw>/sw/umd/out/core/src/compiler/libnvdla_compiler/libnvdla_compiler.so`
3. Main functions of compiler and parser are in `/umd/core/src/compiler/caffe/CaffeParser.cpp`
4. (Not sure) Parser output: A network. Defination of class network (in `/core/include/nvdla/INetwork.h`).
5. Main part of compiler is in `/umd/core/src/compiler/Compiler.cpp`, main function is `Compiler::compileInternal()`


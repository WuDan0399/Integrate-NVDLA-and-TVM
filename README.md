# Integrate-NVDLA-and-TVM(Developing)

## Some notes 
1. Official code for NVDLA software [sw](https://github.com/nvdla/sw). Official document [sw doc](http://nvdla.org/sw/contents.html). 
2.  To compile the umd part in sw, use command `make <path to sw>/sw/umd/out/core/src/compiler/libnvdla_compiler/libnvdla_compiler.so`
3. Main functions of compiler and parser are in `/umd/core/src/compiler/caffe/CaffeParser.cpp`
4. (Not sure) Parser output: A network. Defination of class network (in `/core/include/nvdla/INetwork.h`).
5. Main part of compiler is in `/umd/core/src/compiler/Compiler.cpp`, main function is `Compiler::compileInternal()`
6. For using nvdla_runtime to generate inference output, this will take a long time. If yo run this in a docker container, use `docker commit` to save your container, or after you exit and restart it, all message will lost.

## Current Progress:
1. Successfully compile and run ResNet-101 in docker image.
2. Environment setup.
3. Source code reading of parser.

## Environment Setup (Virtual Platform)
1. Use Ubuntu 14.04 (docker image for my side)
2. Follows http://nvdla.org/vp.html.
  2.1 If fail on VP `make install`, check the issues of vp or sw repo. There is a solution.
  2.2 In step 2.5.1, the demo linux kernel image is in `sw/prebuilt/arm64-linux`, copy the image folder to `vp`, `cp -R <path to sw>/sw/prebuilt/arm64-linux/images <path to vp>/vp/`
  
## Compiler source code reading
```cpp
struct TestAppArgs
{
    std::string project;
    std::string inputPath;
    std::string inputName;
    std::string outputPath;
    std::string testname;
    std::string testArgs;
    std::string prototxt; // This should be folded into testArgs
    std::string caffemodel; // This should be folded into testArgs
    std::string cachemodel; // This should be folded into testArgs

    std::string profileName; // ok here?
    std::string profileFile;
    std::string configtarget;
    std::string calibTable;
    nvdla::QuantizationMode quantizationMode;

    NvU16 numBatches;
    nvdla::DataFormat inDataFormat;
    nvdla::DataType computePrecision;

    std::map<std::string, NvF32> tensorScales;
};
struct TestInfo
{
    // common
    nvdla::IWisdom* wisdom; //wisdom->getcompiler()
    std::string wisdomPath;

    // parse
    std::string modelsPath;
    std::string profilesPath;
    std::string calibTablesPath;

    // runtime
    nvdla::IRuntime* runtime;
    nvdla::ILoadable* compiledLoadable;
    NvU8 *pData;
    std::string inputImagesPath;
    std::string inputLoadablePath;
    std::map<std::string, NvDlaImage*> inputImages;
    std::map<std::string, void *> inputBuffers;
    std::map<std::string, NvDlaImage*> outputImages;
    std::map<std::string, void *> outputBuffers;
    std::vector<SubmitContext*> submits;
    NvU32 timeout;
    NvU16 numBatches; // runtime's point-of-view
    NvU32 numSubmits;
};

NvDlaError compileProfile(const TestAppArgs* appArgs, TestInfo* i);
\* Main Job:
Get the compiler: nvdla::ICompiler* compiler = i->wisdom->getCompiler();
Get target, a string: targetConfigName = appArgs->configtarget;
Determin Profile:
Compile:

*\
```

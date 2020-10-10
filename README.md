# Integrate-NVDLA-and-TVM(Developing)

## Some Notes 
1. Official code for NVDLA software [sw](https://github.com/nvdla/sw). Official document [sw doc](http://nvdla.org/sw/contents.html). 
2.  To compile the umd part in sw, use command `make <path to sw>/sw/umd/out/core/src/compiler/libnvdla_compiler/libnvdla_compiler.so`
3. Main functions of compiler and parser are in `/umd/core/src/compiler/caffe/CaffeParser.cpp`
4. (Not sure) Parser output: A network. Defination of class network (in `/core/include/nvdla/INetwork.h`).
5. Main part of compiler is in `/umd/core/src/compiler/Compiler.cpp`, main function is `Compiler::compileInternal()`
6. For using nvdla_runtime to generate inference output, this will take a long time. If yo run this in a docker container, use `docker commit` to save your container, or after you exit and restart it, all message will lost.
7. Draw caffe model structure: http://ethereon.github.io/netscope/#/editor

## Current Progress:
1. Successfully compile and run ResNet-101.
2. Environment setup.
3. Source code reading of parser.
4. Source code reading of compiler.

##  <span id="vpforbuilt"> Environment Setup (Virtual Platform)</span>
1. Use Ubuntu 14.04.
2. Install following [2. Using the Virtual Simulator](http://nvdla.org/vp.html).

    NOTE: If fail on VP `make install` with `error: ‘template class std::auto_ptr’ is deprecated [-Werror=deprecated-declarations]`, check the [issues #17 of vp](https://github.com/nvdla/vp/issues/17).
  
3. Run the virtual platform following [2.5 Running the Virtual Simulator](http://nvdla.org/vp.html#running-the-virtual-simulator)

    NOTE: In step 2.5.1, the demo linux kernel image is in `sw/prebuilt/arm64-linux`, copy the image folder to `vp`, `cp -R <path to sw>/sw/prebuilt/arm64-linux/images <path to vp>/vp/`

## <span id="vpindocker"> Another Way for Enviroment Setup - virtual platform docker image</span>
The vp docker image official site: [Docker NVDLA VP](https://hub.docker.com/r/nvdla/vp)
1. Start the container

`docker pull nvdla/vp # pull the docker image docker`
`docker run -it -v /home:/home nvdla/vp # create and start the container`

2. Start NVDLA virtual simulator

Inside the container: 
`cd /usr/local/nvdla aarch64_toplevel -c aarch64_nvdla.lua # start the virtual simulator`
\# Login the kernel with account 'root' and password 'nvdla'

Install NVDLA demo kernel driver

After login the kernel: 
`mount -t 9p -o trans=virtio r /mnt # mount pwd` 
`cd /mnt insmod drm.ko # install drm driver`
`insmod opendla.ko # install nvdla driver`

3. Exit NVDLA virtual simulator

ctrl+a x

## How to change and rebuild compiler
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
## How to Run the Whole Process for Model Inference(TODO)
1. Get a caffe model and corresponding prototxt. Currently, we only success on ResNet-101. The model and prototxt can be downloaded here: [ResNet-101 download link](https://1drv.ms/u/s!ArGaVoKpkwjNg0OmwFpdewXh7If_?e=4dxQCa)

2. Compile the model. You can use prebuilt compiler in `sw/prebuilt/x86-ubuntu/`. Use `./nvdla_compiler -h` for futher information.

3. Copy the loadable file (.nvdla) and test image to target (where nvdla_runtime located).

If you use [virtual platform in docker image](#vpforbuilt):
 Copy the loadable file (.nvdla) and image to `vp` folder.

If you use the [virtual platform built on your system](#vpindocker):
 Copy the loadable file (.nvdla) and image to `/usr/local/nvdla` folder.
 
4. Use the `./nvdla_runtime` to run the model.

## Compiler source code reading
### Brief overview of compiler part
![compilerOverview](./compilerOverview.png)
### Usage of execuatble compiler and runtime
```
> ./nvdla_compiler -h
Usage: ./nvdla_compiler [-options] --prototxt <prototxt_file> --caffemodel <caffemodel_file>
where options include:
    -h                                                          print this help message
    -o <outputpath>                                             outputs wisdom files in 'outputpath' directory
    --profile <basic|default|performance|fast-math>             computation profile (default: fast-math)
    --cprecision <fp16|int8>                                    compute precision (default: fp16)
    --configtarget <opendla-full|opendla-large|opendla-small>   target platform (default: nv_full)
    --calibtable <int8 calib file>                              calibration table for INT8 networks (default: 0.00787)
    --quantizationMode <per-kernel|per-filter>                  quantization mode for INT8 (default: per-kernel)
    --batch                                                     batch size (default: 1)
    --informat <ncxhwx|nchw|nhwc>                               input data format (default: nhwc)

> ./nvdla_runtime -h
Usage: ./nvdla_runtime [-options] --loadable <loadable_file>
where options include:
    -h                    print this help message
    -s                    launch test in server mode
    --image <file>        input jpg/pgm file
    --normalize <value>   normalize value for input image
    --mean <value>        comma separated mean value for input image
    --rawdump             dump raw dimg data


```
### Functions
```
NvDlaError compileProfile(const TestAppArgs* appArgs, TestInfo* i);
/* Function:
Get the compiler: nvdla::ICompiler* compiler = i->wisdom->getCompiler();
Get target, a string <opendla-full|opendla-large|opendla-small>: targetConfigName = appArgs->configtarget;
Determine Profile, a string <basic|default|performance|fast-math> : init named profile (basic/default/performance) with default params in its constructor and exit
Compile: use function compile of class compiler.
*/

NvDlaError Compiler::compile(const char *tp_name, const char *target_config_name, ILoadable **peli);
// Function: call compileInternal function

NvDlaError Compiler::compileInternal(const char *tp_name, const char *target_config_name, ILoadable **peli, bool fullCompile);
/* Function: Interface from compile function to compileInternal function */

NvDlaError Compiler::compileInternal(Profile *profile, TargetConfig *target_config, ILoadable **peli, bool fullCompile);
```

### Data Strcutures
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
```

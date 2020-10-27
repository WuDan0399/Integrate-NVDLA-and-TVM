# Working with TVM codebase

## Reading:
1. TVM's BYOC: https://tvm.apache.org/2020/07/15/how-to-bring-your-own-codegen-to-tvm
2. Deploy and Integration: https://tvm.apache.org/docs/deploy/index.html
3. Contribute to TVM: https://tvm.apache.org/docs/contribute/pull_request.html


## Using BYOC
1. https://tvm.apache.org/docs/dev/relay_bring_your_own_codegen.html
2. Relay Arm ® Compute Library Integration:
      1. https://tvm.apache.org/docs/deploy/arm_compute_lib.html
      2. https://discuss.tvm.apache.org/t/rfc-byoc-arm-compute-library-integration/7082
      3. Codebase: https://github.com/apache/incubator-tvm/pull/5915/files
      4. Integrating "add" operation: https://github.com/apache/incubator-tvm/pull/6532/files
      

## Testing code:

We want to test and validate our code using unittests. In general, we can use the following code to test our implementation:
https://tvm.apache.org/docs/contribute/pull_request.html#testing

Dependency:
```pip install --user pytest Cython```

To run all tests:
```
# build tvm
make
# change Python version in the script accordingly
./tests/scripts/task_python_unittest.sh
```
To run any particular tests:
```
# build tvm
make
# replace testfile name with your target file
# All tests reside in tests folder
TVM_FFI=ctypes python3.6 -m pytest -v tests/python/unittest/test_pass_storage_rewrite.py
```
## TO generate JSON file using ARM Compute Library:

### Rebuild TVM compiler

Change set(USE_ARM_COMPUTE_LIB OFF) to set(USE_ARM_COMPUTE_LIB ON) to enable ARM Compute Libraray backend in the build/config.cmake file. Use following commands to rebuild TVM stack. 

```
cd build
cmake ..
make -j4
```

### Code to dump JSON file

```
import tvm
from tvm import relay
from tvm.contrib import util
from tvm.relay.op.contrib import arm_compute_lib

from itertools import zip_longest, combinations
import json
import os
import warnings

import numpy as np


data_type = "float32"
data_shape = (1, 14, 14, 512)
strides = (2, 2)
padding = (0, 0, 0, 0)
pool_size = (2, 2)
layout = "NHWC"
output_shape = (1, 7, 7, 512)

data = relay.var('data', shape=data_shape, dtype=data_type)
out = relay.nn.max_pool2d(data, pool_size=pool_size, strides=strides, layout=layout, padding=padding)
module = tvm.IRModule.from_expr(out)

def extract_acl_modules(module):
    """Get the ACL module(s) from llvm module."""
    return list(filter(lambda mod: mod.type_key == "arm_compute_lib",
                       module.get_lib().imported_modules))

target = "llvm -mtriple=aarch64-linux-gnu -mattr=+neon"
enable_acl = True
params=None
tvm_ops=0
acl_partitions=1

with tvm.transform.PassContext(opt_level=3, disabled_pass=["AlterOpLayout"]):
    if enable_acl:
        module = arm_compute_lib.partition_for_arm_compute_lib(module, params)
    lib = relay.build(module, target=target, params=params)
    acl_modules = extract_acl_modules(lib)
    for mod in acl_modules:
        source = mod.get_source("json")
        codegen = json.loads(source)["nodes"]
        codegen_str = json.dumps(codegen, sort_keys=True, indent=2)
        with open('./tvm/JSON_dump/max_pool2d_readable.json', 'w') as outfile:
            outfile.write(json.dumps(codegen, sort_keys=True, indent=2))
        with open('./tvm/JSON_dump/max_pool2d.json', 'w') as outfile:
            json.dump(codegen, outfile)
```

### Standard format of JSON in ARM's Compute Library

JSON representation has two parts: input and node. Input represents the input for the operation while node represents the attributes of the operations. 

```
Example: Pooling 
   {
   input = {
        "op": "input",
        "name": "",
        "attrs": {"shape": [[list(shape)]], "dtype": [[dtype]]}},
   node = {
        "op": "kernel",
        "name": typef,
        "inputs": [[0, 0, 0]],
        "attrs": {
            "num_inputs": "1",
            "num_outputs": "1",
            "layout": [["NHWC"]],
            "shape": [[list(output_shape)]],
            "dtype": [[dtype]],
            "padding": [[str(p) for p in padding]],
            "strides": [[str(s) for s in strides]],
            "pool_size": [[str(s) for s in sizes]],
            "ceil_mode": [[str(1 if ceil_mode else 0)]]
        },
   }
 ```

## TODO:
1. To define Annotation Rules to describe the supported operators and patterns for NVDLA. 
2. To implement NVDLA Codegen to serialize a Relay graph to a JSON representation.
3. ~~To implement an NVDLA JSON runtime to interpret and execute the serialized JSON graph.~~
4. To combine the necessary NVDLA APIs (including default compiler and runtime) with the TVM default code.
5. To test functional verification of the complete flow and target operators.


## Queries:
1. Can we run single operators like maxpool, conv2d in NVDLA? How does JSON representation look like for such an operator?
2. How can we integrate JSON runtime with NVDLA compiler? How does memory allocation takes place for input/output/weights?
See this: https://tvm.apache.org/docs/dev/relay_bring_your_own_codegen.html#implement-a-customized-runtime

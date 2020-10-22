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

## TODO:
1. To define Annotation Rules to describe the supported operators and patterns for NVDLA. 
2. To implement NVDLA Codegen to serialize a Relay graph to a JSON representation.
3. To implement an NVDLA JSON runtime to interpret and execute the serialized JSON graph.
4. To combine the necessary NVDLA APIs (including default compiler and runtime) with the TVM default code.
5. To test functional verification of the complete flow and target operators.


## Queries:
1. Can we run single operators like maxpool, conv2d in NVDLA? How does JSON representation look like for such an operator?
2. How can we integrate JSON runtime with NVDLA compiler? How does memory allocation takes place for input/output/weights?
See this: https://tvm.apache.org/docs/dev/relay_bring_your_own_codegen.html#implement-a-customized-runtime

# Relay -> NVDLA JSON Format

NVDLA JSON format consists of two parts: "input" and "op". Here, "op" refers to the operators such as Conv2D, MaxPool, ReLu etc. to be offloaded on NVDLA architecture. 
JSON consists of network inputs followed by layer-wise operations. One can look at core tensor operator primitives in available in Relay [here](https://tvm.apache.org/docs/langref/relay_op.html).   

## JSON Format
```yaml
{
  "input": {
    "dtype": [
      [
        "float32"
      ]
    ],
    "shape": [
      [
        [
          1,
          14,
          14,
          512
        ]
      ]
    ]
  },
  "op": {attrs}
}  
```

### Syntax for different operator primitives:

```yaml
"dense": {
  "dtype": [
    [
      "float32"
    ]
  ],
  "num_inputs": "2",
  "num_outputs": "1",
  "out_dtype": [
    [
      ""
    ]
  ],
  "shape": [
    [
      [
        1,
        128
      ]
    ]
  ],
  "units": [
    [
      "128"
    ]
  ]
}
```

```yaml
"relu": {
  "dtype": [
    [
      "float32"
    ]
  ],
  "num_inputs": "1",
  "num_outputs": "1",
  "shape": [
    [
      [
        1,
        128
      ]
    ]
  ]
}
```

```yaml
"softmax": {
  "axis": [
    [
      "-1"
    ]
  ],
  "dtype": [
    [
      "float32"
    ]
  ],
  "num_inputs": "1",
  "num_outputs": "1",
  "shape": [
    [
      [
        1,
        10
      ]
    ]
  ]
}

```

```yaml
"bias_add": {
  "axis": [
    [
      "-1"
    ]
  ],
  "dtype": [
    [
      "float32"
    ]
  ],
  "num_inputs": "2",
  "num_outputs": "1",
  "shape": [
    [
      [
        1,
        128
      ]
    ]
  ]
}
```

```yaml
"batch_flatten": {
  "dtype": [
    [
      "float32"
    ]
  ],
  "num_inputs": "1",
  "num_outputs": "1",
  "shape": [
    [
      [
        1,
        784
      ]
    ]
  ]
}
```




## Example 1

### Relay IR:
```yaml
def @main(%data: Tensor[(1, 14, 14, 512), float32]) -> Tensor[(1, 7, 7, 512), float32] {
  nn.max_pool2d(%data, pool_size=[2, 2], strides=[2, 2], padding=[0, 0, 0, 0], 
  layout="NHWC") /* ty=Tensor[(1, 7, 7, 512), float32] */
}

```

### JSON:
```yaml
{
  "input": {
    "dtype": [
      [
        "float32"
      ]
    ],
    "shape": [
      [
        [
          1,
          14,
          14,
          512
        ]
      ]
    ]
  },
  "max_pool2d_0": {
    "ceil_mode": [
      [
        "0"
      ]
    ],
    "dtype": [
      [
        "float32"
      ]
    ],
    "layout": [
      [
        "NHWC"
      ]
    ],
    "num_inputs": "1",
    "num_outputs": "1",
    "padding": [
      [
        "0",
        "0",
        "0",
        "0"
      ]
    ],
    "pool_size": [
      [
        "2",
        "2"
      ]
    ],
    "shape": [
      [
        [
          1,
          7,
          7,
          512
        ]
      ]
    ],
    "strides": [
      [
        "2",
        "2"
      ]
    ]
  }
}
```

## Example 2

### Relay IR:

```yaml
def @main(%data: Tensor[(1, 1, 28, 28), float32], %fc1_weight: Tensor[(128, 784), float32], %fc1_bias: Tensor[(128), float32], %fc2_weight: Tensor[(64, 128), float32], %fc2_bias: Tensor[(64), float32], %fc3_weight: Tensor[(10, 64), float32], %fc3_bias: Tensor[(10), float32]) -> Tensor[(1, 10), float32] {
  %0 = nn.batch_flatten(%data) /* ty=Tensor[(1, 784), float32] */;
  %1 = nn.dense(%0, %fc1_weight, units=128) /* ty=Tensor[(1, 128), float32] */;
  %2 = nn.bias_add(%1, %fc1_bias, axis=-1) /* ty=Tensor[(1, 128), float32] */;
  %3 = nn.relu(%2) /* ty=Tensor[(1, 128), float32] */;
  %4 = nn.dense(%3, %fc2_weight, units=64) /* ty=Tensor[(1, 64), float32] */;
  %5 = nn.bias_add(%4, %fc2_bias, axis=-1) /* ty=Tensor[(1, 64), float32] */;
  %6 = nn.relu(%5) /* ty=Tensor[(1, 64), float32] */;
  %7 = nn.dense(%6, %fc3_weight, units=10) /* ty=Tensor[(1, 10), float32] */;
  %8 = nn.bias_add(%7, %fc3_bias, axis=-1) /* ty=Tensor[(1, 10), float32] */;
  nn.softmax(%8) /* ty=Tensor[(1, 10), float32] */
}
```

### JSON:
```yaml
{'input': {'dtype': [['float32']], 'shape': [[[1, 1, 28, 28]]]}, 'batch_flatten_0': {'num_outputs': '1', 'num_inputs': '1', 'dtype': [['float32']], 'shape': [[[1, 784]]]}, 'dense_0': {'num_outputs': '1', 'num_inputs': '2', 'out_dtype': [['']], 'dtype': [['float32']], 'units': [['128']], 'shape': [[[1, 128]]]}, 'bias_add_0': {'num_outputs': '1', 'axis': [['-1']], 'shape': [[[1, 128]]], 'dtype': [['float32']], 'num_inputs': '2'}, 'relu_0': {'num_outputs': '1', 'num_inputs': '1', 'dtype': [['float32']], 'shape': [[[1, 128]]]}, 'dense_1': {'num_outputs': '1', 'num_inputs': '2', 'out_dtype': [['']], 'dtype': [['float32']], 'units': [['64']], 'shape': [[[1, 64]]]}, 'bias_add_1': {'num_outputs': '1', 'axis': [['-1']], 'shape': [[[1, 64]]], 'dtype': [['float32']], 'num_inputs': '2'}, 'relu_1': {'num_outputs': '1', 'num_inputs': '1', 'dtype': [['float32']], 'shape': [[[1, 64]]]}, 'dense_2': {'num_outputs': '1', 'num_inputs': '2', 'out_dtype': [['']], 'dtype': [['float32']], 'units': [['10']], 'shape': [[[1, 10]]]}, 'bias_add_2': {'num_outputs': '1', 'axis': [['-1']], 'shape': [[[1, 10]]], 'dtype': [['float32']], 'num_inputs': '2'}, 'softmax_2': {'num_outputs': '1', 'axis': [['-1']], 'shape': [[[1, 10]]], 'dtype': [['float32']], 'num_inputs': '1'}}
```


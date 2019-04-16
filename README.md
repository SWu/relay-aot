# relay-aot

An experimental ahead of time compiler for Relay.

The ahead of time compiler enables the execution of Relay code without
requiring a framework's interpreter written in C++ or Python.

The removal of framework and interpretation overhead combined
with optimized operators produced by TVM's operators dramatically
reduces execution time. Additionally, the compiler produces a single
binary which only depends on TVM, simplifying the deployment story.
This repository contains an external library which  implements ahead of time
compilation for Relay.
The current approach is a proof of concept which lowers Relay to C++, and relies on a
C++ compiler such as `gcc` or `clang`  to produce an executable.

The ahead of time compiler comes as a standalone library which exposes a
primitive `compile` function which compiles a `relay.Function`
into a Python closure which wraps the compiled native code.

The compiler's design is straight forward it lowers functions into a
small C++-like IR, and generates a C++ program which can be compiled
and dynamically linked. We extract the corresponding symbol from the
dynamic library, and wrap it as a Python closure.

You can see an example below:

```
import numpy as np
import tvm
from tvm.relay import Module, GlobalVar, Function
from aot import compile

def double_example():
    # Declare a Relay module.
    mod = Module()

    # Implement the double function.
    x = var('x', shape=())
    double = GlobalVar('double')
    mod[double] = Function([x], x + x)

    # Generate a function which calls double twice.
    x = var('x', shape=())
    f = Function([x], double(double(x)))
    # Compile the function.
    cfunc = compile(f, mod)

    a = tvm.nd.array(np.array(1.5, dtype='float32'))
    output = cfunc(a).asnumpy() // array(6.)
```

You can test to ensure you setup the ahead of time compiler correctly
adding it to your `PYTHONPATH`, and running:
```
python3 examples/readme_ex.py
```

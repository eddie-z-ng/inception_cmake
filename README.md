### CMake Tensorflow Serving Client
These are my hacks around getting a Tensorflow Serving client (using the tutorial `inception_client.cc`) in C++ built via CMake. There's a few changes in the CMake submodules to let me invoke the protobuf compiler to support the *.proto file layout that exists in the `tensorflow/serving` repository. This checks out the `serving` submodule to the official repository and moves the CMakeLists.txt file out. This assumes the following have been installed:

1. Protobuf (I'm using `libprotoc 3.4.0`)
2. gRPC (I'm using `libgrpc.so.5.0.0`)
3. Tensorflow installed via [tensorflow_cc](https://github.com/FloopCZ/tensorflow_cc). I used `TensorflowCC::Shared` but `TensorflowCC::Static` should also work for this example


### Build
```bash
mkdir build && cd build
cmake ..
make
```

# GlimmerWorksProtobuf

### Compile documentation

Fedora

```
sudo dnf5 install protobuf-compiler
```
Compile c++ to the output directory

```
mkdir -p build
protoc --cpp_out=./build saves/*.proto
```

### As a cmake module

```
cmake -S . -B cmake-build-debug -G Ninja
cd cmake-build-debug
ninja
```
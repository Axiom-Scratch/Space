# procedural_planet

Minimal procedural planet project built with CMake.

## Build

```sh
cmake -S . -B build -G Ninja -DCMAKE_BUILD_TYPE=Debug
cmake --build build
```

## Run

```sh
./build/Sandbox
```

## Options

- `CMAKE_BUILD_TYPE=Debug|Release|RelWithDebInfo`
- `AXIOM_ENABLE_ASSERTS=ON|OFF`
- `AXIOM_ENABLE_LOGGING=ON|OFF`
- `AXIOM_ENABLE_ASAN=ON|OFF`
- `AXIOM_ENABLE_UBSAN=ON|OFF`
- `AXIOM_ENABLE_TSAN=ON|OFF`
- `AXIOM_ENABLE_TRACY=ON|OFF`
- `AXIOM_ENABLE_LTO=ON|OFF`
- `AXIOM_WARNINGS_AS_ERRORS=ON|OFF`

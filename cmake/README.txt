Build
Basic
rm -rf build
cmake -B build -G Ninja
cmake --build build
Debug
cmake -B build -DCMAKE_BUILD_TYPE=Debug
cmake --build build
Release
cmake -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build
Debug + Sanitizers (recommended)
cmake -B build \
  -DCMAKE_BUILD_TYPE=Debug \
  -DAXIOM_ENABLE_ASAN=ON \
  -DAXIOM_ENABLE_UBSAN=ON

cmake --build build
Strict Warnings
cmake -B build -DAXIOM_WARNINGS_AS_ERRORS=ON
Run

If you have an exe

./build/Sandbox



  1. cmake -S . -B build-clang -DCMAKE_C_COMPILER=clang -DCMAKE_CXX_COMPILER=clang++
     Configure succeeded and regenerated the compile database for clang/clangd.
  2. cmake --build build-clang -j4
     Build succeeded: Built target app.
  3. compile_commands.json
     Confirmed it now points to build-clang/compile_commands.json, and the src/main.cpp command includes both -I/home/axiom-scratch/Desktop/Planet/external/glad/include and -I/home/axiom-scratch/
     Desktop/Planet/external/glfw/include.
Token usage: total=63,476 input=48,553 (+ 962,688 cached) output=14,923 (reasoning 8,916)
To continue this session, run codex resume 019d3d2a-57b2-7ce1-9d39-c65707c11538

~/Desktop/Planet master* 7m 34s
❯ 
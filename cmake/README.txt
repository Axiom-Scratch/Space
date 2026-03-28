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

# Falcor 7.0 + tiny-cuda-nn

This is a tutorial on how to compile [tiny-cuda-nn](https://github.com/NVlabs/tiny-cuda-nn) with [Falcor](https://github.com/NVIDIAGameWorks/Falcor), in the aim of using deep learning in computer graphics. 

I have searched everywhere on-line only to find there is not a single open source project teaching how to use a neural network in Falcor, so I decided to create this tutorial for the convenience of more graphics/deep learning researchers to make use of both great tools.



## Clone the repository

You can clone the repository in two ways:

#### Directly clone my repo:

```bash
git clone --recursive https://github.com/yijie21/Falcor-tiny-cuda-nn.git
```

#### Or, clone the two original repos:

```bash
git clone --recursive https://github.com/NVIDIAGameWorks/Falcor
cd external
git clone --recursive https://github.com/NVlabs/tiny-cuda-nn
```



## CMake Modifications

The integration of tiny-cuda-nn in Falcor requires the following cmake modifications if you **clone the repository by cloning the two original repos**.

1. Add **CUDA** language in the root `Falcor/CMakeLists.txt` project setting:

   ```cmake
   project(Falcor
       DESCRIPTION "Falcor Realtime Rendering Framework"
       LANGUAGES CXX C CUDA
   )
   ```

2. Add cmake configuration for tiny-cuda-nn in `Falcor/external/CMakeLists.txt`

   ```cmake
   # step 1: add the 4 lines below after vulkan-headers for tiny-cuda-nn compiling
   # tiny-cuda-nn
   set(TCNN_BUILD_BENCHMARK OFF)
   set(TCNN_BUILD_EXAMPLES OFF)
   set(TCNN_ALLOW_CUBLAS_CUSOLVER OFF)
   add_subdirectory(tiny-cuda-nn)
   
   # step 2: comment the cmake configuration below for fmt because there is already a fmt dependency in tiny-cuda-nn/dependencies
   # message(STATUS "Configure fmt")
   # add_subdirectory(fmt)
   # set_target_properties(fmt PROPERTIES
   #     POSITION_INDEPENDENT_CODE ON
   #     FOLDER "Libraries"
   # )
   
   # step 3: add external includes "tiny-cuda-nn/dependencies" for Falcor because Falcor needs to include fmt for successful compilation
   # external_includes
   add_library(external_includes INTERFACE)
   target_include_directories(external_includes INTERFACE include tiny-cuda-nn/dependencies)
   ```



## Create a new pass to test tiny-cuda-nn

1. Open the terminal in the root folder and type the following code to create a new pass(the new pass is in `Falcor/Source/RenderPasses/NeuralPass`):

   ```bash
   .\tools\make_new_render_pass.bat NeuralPass
   ```

2. Create a `main.cu` file in `NeuralPass` folder and write the following code:

   ```c++
   #include <tiny-cuda-nn/common_device.h>
   #include <tiny-cuda-nn/config.h>
   #include <iostream>
   
   int main(int argc, char** argv) {
     std::cout << "Hello World!" << std::endl;
     return 0;
   }
   ```

3. Configure for the `main.cu` in `NeuralPass/CMakeLists.txt`

   ```cmake
   # step 1: add main.cu in target_sources
   # step 2: add tiny-cuda-nn in target_link_libraries
   add_plugin(NeuralPass)
   
   target_sources(NeuralPass PRIVATE
       NeuralPass.cpp
       NeuralPass.h
       main.cu
   )
   
   target_link_libraries(NeuralPass PRIVATE tiny-cuda-nn)
   
   target_source_group(NeuralPass "RenderPasses")
   
   ```



## Compile the project

Just follow the compiling instructions in Falcor; or follow the following steps below:

1. Open terminal in the root folder, and type the following code for cmake configuration(make sure you have VS2022 installed):

   ```bash
   .\setup_vs2022.bat
   ```

2. [Important] Open `Falcor/build/windows-vs2022/Source` with VSCode, search for `<TreatWarningAsError>true` and replace them all with  `<TreatWarningAsError>false`. Otherwise, visual studio 2022 will report many errors related to encoding.

3. Type the following code to compile:

   ```bash
    cmake --build build/windows-vs2022 -j8
   ```

4. [Important] If an error of `"MIN_GPU_ARCH= XX0 must bound __CUDA_ARCH__=860 from below, but doesn't."` happened, just comment the lines 81-83 in `Falcor/external/tiny-cuda-nn/include/tiny-cuda-nn/common.h` to solve the error:

   ```c++
   // #if defined(__CUDA_ARCH__)
   // static_assert(__CUDA_ARCH__ >= TCNN_MIN_GPU_ARCH * 10, "MIN_GPU_ARCH=" STR(TCNN_MIN_GPU_ARCH) "0 must bound __CUDA_ARCH__=" STR(__CUDA_ARCH__) " from below, but doesn't.");
   // #endif
   ```



## Pay Special Attention to 

- Never include a header file in tiny-cuda-nn in a `.c/.cpp` file, this will cause errors.
- There are some shared parameter declarations in both Falcor and cuda, like `float3/uint2`, so you'd better use `Falcor::float3/uint2` in a Falcor `.cpp` file to avoid ambiguity.



Up to now, Falcor will be successfully compiled with tiny-cuda-nn, enjoy developing with both powerful tools! If you have any questions, please let me know in the `issue`!


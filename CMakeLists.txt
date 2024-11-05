cmake_minimum_required(VERSION 3.18)

project(PolycubeHexMesher LANGUAGES CXX CUDA)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CUDA_STANDARD 17)
set(CMAKE_CUDA_STANDARD_REQUIRED ON)
set(HDF5_USE_STATIC_LIBRARIES OFF)

# Allow custom CMakeLists.txt
include(${PROJECT_SOURCE_DIR}/CMakeCustomLists.txt OPTIONAL)

# assumes find_package(CUDAToolkit) was already done
if(
    PYTORCH_VERSION VERSION_GREATER_EQUAL 2.5.0 AND
    PYTORCH_CUDA_VERSION VERSION_GREATER_EQUAL 12
)
    message(STATUS "PyTorch NVTX headers workaround: Yes")
    # only do this if nvToolsExt is not defined and CUDA::nvtx3 exists
    if(NOT TARGET CUDA::nvToolsExt AND TARGET CUDA::nvtx3)
        add_library(CUDA::nvToolsExt INTERFACE IMPORTED)
        # ensure that PyTorch is told to use NVTX3 headers
        target_compile_definitions(
            CUDA::nvToolsExt INTERFACE
            TORCH_CUDA_USE_NVTX3
        )
        target_link_libraries(CUDA::nvToolsExt INTERFACE CUDA::nvtx3)
    endif()
else()
    message(STATUS "PyTorch NVTX headers workaround: No")
endif()
# find Torch C++ as usual
find_package(Torch CONFIG)

# Vulkan
find_package(Vulkan REQUIRED)

# OpenMP
find_package(OpenMP REQUIRED)

if (NOT CMAKE_BUILD_TYPE)
    message(STATUS "No build type selected; default to release.")
    set(CMAKE_BUILD_TYPE "Release")
endif ()

if (CMAKE_CXX_COMPILER_ID MATCHES "Clang")
    set(cxx_warning_flags "-Wall" "-ferror-limit=1")
elseif (CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
    set(cxx_warning_flags "-Wall")
elseif (CMAKE_CXX_COMPILER_ID STREQUAL "MSVC")
    add_compile_options (/permissive-)
    set(cxx_warning_flags "/W0")
endif()

add_subdirectory(external)
add_subdirectory(vkoo)
add_subdirectory(geomlib)
add_subdirectory(hex)
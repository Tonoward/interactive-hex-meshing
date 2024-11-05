# Set CUDA standard and architectures
set(CMAKE_CUDA_STANDARD 17)
set(CMAKE_CUDA_STANDARD_REQUIRED ON)
set(CMAKE_CUDA_ARCHITECTURES 75)

file(GLOB geomlib_src *.cpp io/*.cpp *.cu autograd/*.cpp)

add_library(geomlib ${geomlib_src})

# Set CUDA-specific properties
set_target_properties(geomlib PROPERTIES
    CUDA_SEPARABLE_COMPILATION ON
    CUDA_ARCHITECTURES 86)

target_include_directories(geomlib PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/../")

# Linking libraries using the keyword signature
target_link_libraries(geomlib PUBLIC 
    eigen 
    CUDA::cudart 
    CUDA::cusolver 
    CUDA::cupti 
    ${TORCH_LIBRARIES} 
    OpenMP::OpenMP_CXX 
    spdlog)

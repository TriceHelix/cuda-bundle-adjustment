cmake_minimum_required(VERSION 3.15)

cmake_policy(SET CMP0091 NEW)

project(CudaBundleAdjustment)

include(CMakePrintHelpers)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_EXTENSIONS OFF)

set(CMAKE_CUDA_STANDARD 17)
set(CMAKE_CUDA_STANDARD_REQUIRED ON)

option(ENABLE_SAMPLES "Build samples" ON)
option(WITH_G2O       "Build sample with g2o (g2o needs to be installed)" OFF)
option(USE_FLOAT32    "Use 32bit float in internal floating-point operations (default is 64bit float)" OFF)

option(BUILD_SHARED_LIBS "build shared libraries" ON)
cmake_print_variables(BUILD_SHARED_LIBS)

add_subdirectory(src)

if(ENABLE_SAMPLES)
	add_subdirectory(samples/sample_ba_from_file)
	if(WITH_G2O)
		add_subdirectory(samples/sample_comparison_with_g2o)
	endif()
endif()

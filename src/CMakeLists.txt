cmake_minimum_required(VERSION 3.15)

cmake_policy(SET CMP0091 NEW)

project(cuda_bundle_adjustment LANGUAGES CUDA CXX)

if(NOT DEFINED CMAKE_INSTALL_PREFIX)
  set(CMAKE_INSTALL_PREFIX ${CMAKE_CURRENT_BINARY_DIR}/install)
endif()

set(CUBA_INCLUDE_DIR ${CMAKE_SOURCE_DIR}/include)
include_directories(${CUBA_INCLUDE_DIR})

# Find CUDA Toolkit
find_package(CUDAToolkit MODULE REQUIRED)

# Set CUDA architectures
set(CMAKE_CUDA_ARCHITECTURES 50 52 60 62 70 72 75 80 86 CACHE STRING "CUDA architectures")

# Build shared/static lib
set(CUBA_LIB_TYPE SHARED)
if(NOT ${BUILD_SHARED_LIBS})
  set(CUBA_LIB_TYPE STATIC)
endif()

# Runtime Library
set(CMAKE_CUDA_RUNTIME_LIBRARY Static)

# Eigen
find_package(Eigen3 REQUIRED)
include_directories(${EIGEN3_INCLUDE_DIR})

# Flags for each platform
if (CMAKE_COMPILER_IS_GNUCXX)
	set(CMAKE_CXX_FLAGS "-O3 -Wall")
endif()

if(WIN32)
	#add_compile_options(/wd4819 /wd4244)
	add_definitions(-D_SILENCE_CXX17_NEGATORS_DEPRECATION_WARNING)
endif()

if (USE_FLOAT32)
	add_definitions(-DUSE_FLOAT32)
endif()

file(GLOB srcs ./*.cpp ./*.cu ./*.h ${CUBA_INCLUDE_DIR}/*.h)
add_library(cuda_bundle_adjustment ${CUBA_LIB_TYPE} ${srcs})

# CUDA compile options
target_compile_options(cuda_bundle_adjustment PRIVATE $<$<COMPILE_LANGUAGE:CUDA>:
  --generate-line-info
  --use_fast_math
>)

# Link against CUDA libraries
target_link_libraries(cuda_bundle_adjustment ${CUDA_cusparse_LIBRARY} ${CUDA_cusolver_LIBRARY})

install(
	TARGETS cuda_bundle_adjustment
	ARCHIVE DESTINATION ${CMAKE_INSTALL_PREFIX}/lib
	LIBRARY DESTINATION ${CMAKE_INSTALL_PREFIX}/lib
	RUNTIME DESTINATION ${CMAKE_INSTALL_PREFIX}/bin
)

install(
	DIRECTORY ${CUBA_INCLUDE_DIR}
	DESTINATION ${CMAKE_INSTALL_PREFIX}
	FILES_MATCHING PATTERN "*.h" PATTERN "*.hpp"
)

# SPDX-FileCopyrightText: Copyright (c) 2023 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
# SPDX-License-Identifier: MIT
#
# Permission is hereby granted, free of charge, to any person obtaining a
# copy of this software and associated documentation files (the "Software"),
# to deal in the Software without restriction, including without limitation
# the rights to use, copy, modify, merge, publish, distribute, sublicense,
# and/or sell copies of the Software, and to permit persons to whom the
# Software is furnished to do so, subject to the following conditions:
#
# The above copyright notice and this permission notice shall be included in
# all copies or substantial portions of the Software.
#
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL
# THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING
# FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER
# DEALINGS IN THE SOFTWARE.

cmake_minimum_required(VERSION 2.8.12)
project(bevfusion)

set(CMAKE_BUILD_TYPE "Release")
# set(CMAKE_BUILD_TYPE "Debug")
set(arch ${CMAKE_HOST_SYSTEM_PROCESSOR})

find_package(CUDA REQUIRED)

# For cuda-12
# set(CMAKE_CXX_FLAGS_RELEASE "-std=c++17 -Wextra -Wall -Wno-missing-field-initializers -Wno-deprecated-declarations -O3 -DENABLE_TEXT_BACKEND_STB")
# set(CMAKE_CXX_FLAGS_DEBUG   "-std=c++17 -O0 -g -DENABLE_TEXT_BACKEND_STB")

set(CMAKE_CXX_FLAGS_RELEASE "-std=c++14 -Wextra -Wall -Wno-missing-field-initializers -Wno-deprecated-declarations -O3 -DENABLE_TEXT_BACKEND_STB")
set(CMAKE_CXX_FLAGS_DEBUG   "-std=c++14 -O0 -g -DENABLE_TEXT_BACKEND_STB")

set(CUDA_NVCC_FLAGS_RELEASE "-Werror=all-warnings -Xcompiler -std=c++14,-Wextra,-Wall,-Wno-deprecated-declarations,-O3 -DENABLE_TEXT_BACKEND_STB")
set(CUDA_NVCC_FLAGS_DEBUG   "-std=c++14 -O0 -g -DENABLE_TEXT_BACKEND_STB")

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++14")
set(CUDA_NVCC_FLAGS "${CUDA_NVCC_FLAGS} --std c++14")

# TODO(josh): which ones do we want
set(CUDA_NVCC_FLAGS ${CUDA_NVCC_FLAGS}
  -gencode arch=compute_89,code=compute_89
)

set(CMAKE_POSITION_INDEPENDENT_CODE ON)

include_directories(
  $ENV{CUDA_Inc}
  $ENV{TensorRT_Inc}
  ../dependencies/stb
  ../dependencies/pybind11/include
  ../libraries/cuOSD/src
  ../libraries/3DSparseConvolution/libspconv/include
  src/
)

link_directories(
  $ENV{CUDA_Lib}
  $ENV{TensorRT_Lib}
  ../libraries/3DSparseConvolution/libspconv/lib/${arch}
  build
)

file(GLOB_RECURSE CORE_FILES
  src/bevfusion/*.cu
  src/bevfusion/*.cpp
  src/common/tensor.cu
  src/common/tensorrt.cpp
)

file(GLOB_RECURSE VISUALIZE_FILES
  ../libraries/cuOSD/src/*.cpp
  ../libraries/cuOSD/src/*.cu
  src/common/visualize.cu
)

####################### libbevfusion_core.so ##########################
cuda_add_library(bevfusion_core STATIC
  ${CORE_FILES}
)

cuda_add_library(bevfusion_vis STATIC
  ${VISUALIZE_FILES}
)

target_link_libraries(bevfusion_core
  libcudart.so
  libcublasLt.so
  libnvinfer.so
  libspconv.so
  libprotobuf.so.17
  libnvinfer_plugin.so
)
#######################################################################

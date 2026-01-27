# Copyright (c) Meta Platforms, Inc. and affiliates.
# All rights reserved.
# Copyright 2025 Arm Limited and/or its affiliates.
#
# SPDX-License-Identifier: Apache-2.0

cmake_minimum_required(VERSION 3.20)

find_package(Zephyr REQUIRED HINTS $ENV{ZEPHYR_BASE})
project(executorch_executor_runner)

# -----------------------------------------------------------------------------
# AI Layer Configuration
# -----------------------------------------------------------------------------
# The ai_layer contains pre-built ExecuTorch libraries and headers generated
# by the Docker build workflow. This simplifies the build by using pre-compiled
# artifacts rather than building ExecuTorch from source.

set(AI_LAYER_DIR ${CMAKE_CURRENT_SOURCE_DIR}/ai_layer)
set(AI_LAYER_ENGINE_DIR ${AI_LAYER_DIR}/engine)
set(AI_LAYER_LIB_DIR ${AI_LAYER_ENGINE_DIR}/lib)
set(AI_LAYER_INCLUDE_DIR ${AI_LAYER_ENGINE_DIR}/include)
set(AI_LAYER_MODEL_DIR ${AI_LAYER_DIR}/model)

# Verify ai_layer exists
if(NOT EXISTS ${AI_LAYER_LIB_DIR}/libexecutorch_core.a)
  message(FATAL_ERROR 
    "AI Layer libraries not found at ${AI_LAYER_LIB_DIR}.\n"
    "Please run the Docker build workflow first:\n"
    "  docker build -f zephyr/.docker/Dockerfile -t executorch-arm-builder .\n"
    "  docker run --rm -v \$(pwd):/workspace2 executorch-arm-builder:latest /workspace2/executorch/zephyr/scripts/local_workflow.sh"
  )
endif()

# -----------------------------------------------------------------------------
# Include Directories
# -----------------------------------------------------------------------------
# Add all necessary include paths from the ai_layer engine

target_include_directories(app PRIVATE
  ${AI_LAYER_INCLUDE_DIR}
  ${AI_LAYER_INCLUDE_DIR}/executorch
  ${AI_LAYER_INCLUDE_DIR}/executorch/runtime
  ${AI_LAYER_INCLUDE_DIR}/executorch/runtime/core
  ${AI_LAYER_INCLUDE_DIR}/executorch/runtime/executor
  ${AI_LAYER_INCLUDE_DIR}/executorch/extension
  ${AI_LAYER_INCLUDE_DIR}/executorch/extension/data_loader
  ${AI_LAYER_INCLUDE_DIR}/executorch/extension/memory_allocator
  ${AI_LAYER_INCLUDE_DIR}/executorch/extension/runner_util
  ${AI_LAYER_INCLUDE_DIR}/executorch/util
  ${AI_LAYER_INCLUDE_DIR}/executorch/schema
  ${AI_LAYER_INCLUDE_DIR}/executorch/devtools
  ${AI_LAYER_INCLUDE_DIR}/executorch/devtools/etdump
  ${AI_LAYER_INCLUDE_DIR}/executorch/runtime/core/portable_type/c10
  ${AI_LAYER_INCLUDE_DIR}/executorch/kernels
  ${AI_LAYER_INCLUDE_DIR}/executorch/kernels/portable
  ${AI_LAYER_INCLUDE_DIR}/executorch/backends
  ${AI_LAYER_INCLUDE_DIR}/executorch/backends/cortex_m
  ${AI_LAYER_INCLUDE_DIR}/executorch/backends/cortex_m/ops
  ${AI_LAYER_INCLUDE_DIR}/executorch/examples/arm/executor_runner
  ${AI_LAYER_INCLUDE_DIR}/third-party
  ${AI_LAYER_INCLUDE_DIR}/third-party/flatbuffers
  ${AI_LAYER_INCLUDE_DIR}/third-party/flatbuffers/include
  ${AI_LAYER_INCLUDE_DIR}/third-party/flatcc
  ${AI_LAYER_INCLUDE_DIR}/third-party/flatcc/include
  ${AI_LAYER_MODEL_DIR}
  ${CMAKE_CURRENT_SOURCE_DIR}/src
)

# -----------------------------------------------------------------------------
# Import Pre-built Libraries
# -----------------------------------------------------------------------------
# Import all static libraries from the ai_layer engine

# Core ExecuTorch libraries (always required)
add_library(executorch_core STATIC IMPORTED)
set_target_properties(executorch_core PROPERTIES
  IMPORTED_LOCATION ${AI_LAYER_LIB_DIR}/libexecutorch_core.a
)

add_library(executorch STATIC IMPORTED)
set_target_properties(executorch PROPERTIES
  IMPORTED_LOCATION ${AI_LAYER_LIB_DIR}/libexecutorch.a
)

# Portable kernels and ops
add_library(portable_kernels STATIC IMPORTED)
set_target_properties(portable_kernels PROPERTIES
  IMPORTED_LOCATION ${AI_LAYER_LIB_DIR}/libportable_kernels.a
)

add_library(portable_ops_lib STATIC IMPORTED)
set_target_properties(portable_ops_lib PROPERTIES
  IMPORTED_LOCATION ${AI_LAYER_LIB_DIR}/libportable_ops_lib.a
)

# Quantized kernels and ops
add_library(quantized_kernels STATIC IMPORTED)
set_target_properties(quantized_kernels PROPERTIES
  IMPORTED_LOCATION ${AI_LAYER_LIB_DIR}/libquantized_kernels.a
)

add_library(quantized_ops_lib STATIC IMPORTED)
set_target_properties(quantized_ops_lib PROPERTIES
  IMPORTED_LOCATION ${AI_LAYER_LIB_DIR}/libquantized_ops_lib.a
)

# Cortex-M optimized kernels (optional, for Cortex-M targets)
if(EXISTS ${AI_LAYER_LIB_DIR}/libcortex_m_kernels.a)
  add_library(cortex_m_kernels STATIC IMPORTED)
  set_target_properties(cortex_m_kernels PROPERTIES
    IMPORTED_LOCATION ${AI_LAYER_LIB_DIR}/libcortex_m_kernels.a
  )

  add_library(cortex_m_ops_lib STATIC IMPORTED)
  set_target_properties(cortex_m_ops_lib PROPERTIES
    IMPORTED_LOCATION ${AI_LAYER_LIB_DIR}/libcortex_m_ops_lib.a
  )
endif()

# Ethos-U delegate (optional, for NPU acceleration)
if(EXISTS ${AI_LAYER_LIB_DIR}/libexecutorch_delegate_ethos_u.a)
  add_library(executorch_delegate_ethos_u STATIC IMPORTED)
  set_target_properties(executorch_delegate_ethos_u PROPERTIES
    IMPORTED_LOCATION ${AI_LAYER_LIB_DIR}/libexecutorch_delegate_ethos_u.a
  )
endif()

# -----------------------------------------------------------------------------
# Application Sources
# -----------------------------------------------------------------------------

set(APP_SOURCES
  src/arm_executor_runner.cpp
  src/arm_memory_allocator.cpp
  src/posix_stub.cpp
  src/random_ops_stubs.cpp
  src/ethos_u_stub.cpp
  src/register_quantized_ops.cpp
)

target_sources(app PRIVATE ${APP_SOURCES})

# -----------------------------------------------------------------------------
# Compiler Flags
# -----------------------------------------------------------------------------

target_compile_options(app PRIVATE
  -Wall
  -Wno-switch
  -Wno-float-conversion
  -Wno-double-promotion
  -ffunction-sections
  -fdata-sections
  -fno-pic
  -fno-pie
)

# Memory pool configuration from Kconfig
if(DEFINED CONFIG_EXECUTORCH_METHOD_ALLOCATOR_POOL_SIZE)
  target_compile_definitions(app PRIVATE
    ET_ARM_METHOD_ALLOCATOR_POOL_SIZE=${CONFIG_EXECUTORCH_METHOD_ALLOCATOR_POOL_SIZE}
  )
endif()

if(DEFINED CONFIG_EXECUTORCH_TEMP_ALLOCATOR_POOL_SIZE)
  target_compile_definitions(app PRIVATE
    ET_ARM_BAREMETAL_SCRATCH_TEMP_ALLOCATOR_POOL_SIZE=${CONFIG_EXECUTORCH_TEMP_ALLOCATOR_POOL_SIZE}
  )
endif()

# ExecuTorch compile definitions for bare-metal/RTOS builds
target_compile_definitions(app PRIVATE
  # Skip cmake_macros.h include - we provide our own definitions
  C10_USING_CUSTOM_GENERATED_MACROS
  # ExecuTorch mobile/embedded build
  C10_MOBILE=1
)

# -----------------------------------------------------------------------------
# Link Libraries
# -----------------------------------------------------------------------------
# Link order matters for static libraries - dependencies must come after dependents

target_link_libraries(app PRIVATE
  # Core ExecuTorch runtime
  executorch
  executorch_core
  
  # Quantized operators (used by most models)
  quantized_ops_lib
  quantized_kernels
  
  # Portable operators (fallback for non-delegated ops)
  portable_ops_lib
  portable_kernels
)

# Add Cortex-M optimized kernels if available and targeting Cortex-M
# Note: These require CMSIS-NN library, so only link if it's available
if(CONFIG_CPU_CORTEX_M AND TARGET cortex_m_ops_lib AND TARGET CMSIS_NN)
  target_link_libraries(app PRIVATE
    cortex_m_ops_lib
    cortex_m_kernels
    CMSIS_NN
  )
endif()

# Add Ethos-U delegate if available
# The delegate contains static initializers that register the backend with ExecuTorch
if(EXISTS ${AI_LAYER_LIB_DIR}/libexecutorch_delegate_ethos_u.a)
  # Link the delegate library 
  target_link_libraries(app PRIVATE ${AI_LAYER_LIB_DIR}/libexecutorch_delegate_ethos_u.a)
  
  # Link the object files directly to ensure static constructor is included
  # The static constructor _GLOBAL__sub_I_EthosUBackend.cpp registers the backend
  if(EXISTS ${AI_LAYER_LIB_DIR}/EthosUBackend.cpp.obj)
    target_link_libraries(app PRIVATE 
      ${AI_LAYER_LIB_DIR}/EthosUBackend.cpp.obj
      ${AI_LAYER_LIB_DIR}/VelaBinStream.cpp.obj
    )
  endif()
  
  # Link Zephyr's Ethos-U driver if available
  if(TARGET ethosu_core_driver)
    target_link_libraries(app PRIVATE ethosu_core_driver)
  endif()
endif()

# -----------------------------------------------------------------------------
# Debug Output
# -----------------------------------------------------------------------------

get_target_property(APP_LIBS app LINK_LIBRARIES)
message(STATUS "ExecuTorch app linked libraries: ${APP_LIBS}")

# Copyright (c) Meta Platforms, Inc. and affiliates.

cmake_minimum_required(VERSION 3.20)

# our build script will first create a src/ dir where all source code will exist
file (GLOB dynolog_src "*.h" "*.cpp")

# Remove main from library, only needed for exec.
list(REMOVE_ITEM dynolog_src "dynolog/src/Main.cc")
add_library(dynolog_lib ${dynolog_src})

if(USE_ODS_GRAPH_API)
  target_compile_options(dynolog_lib PUBLIC "-DUSE_GRAPH_ENDPOINT")
endif()

include_directories(${PROJECT_SOURCE_DIR}/third_party/range-v3/include)

target_link_libraries(dynolog_lib PUBLIC Monitor)
target_link_libraries(dynolog_lib PUBLIC BuiltinMetrics)

add_subdirectory(rpc)

add_subdirectory(ipcfabric)
target_link_libraries(dynolog_lib PUBLIC dynolog_ipcfabric_lib)

# depends on ipcfabric
add_subdirectory(tracing)
target_link_libraries(dynolog_lib PUBLIC dynolog_ipcmonitor_lib)

add_subdirectory(gpumon)
target_link_libraries(dynolog_lib PUBLIC dynolog_dcgm_lib "-ldl")

add_subdirectory(metric_frame)

add_executable(dynolog Main.cpp)
target_link_libraries(dynolog PRIVATE dynolog_lib dynolog_rpc_lib)

# Copyright 2020 Google LLC
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#      http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.


cmake_minimum_required(VERSION 3.10)

project(stadia-performance-layers CXX)

set(CMAKE_CXX_STANDARD 17)

# Disable C++ exceptions.
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fno-exceptions -fno-rtti")

set(FILESYSTEM_LIB_NAME stdc++fs)

# Abseil requires PIC.
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fPIC")

# Add external dependencies.
add_subdirectory(third_party)

if(DEFINED VULKAN_HEADERS_INSTALL_DIR)
  find_path(VulkanHeaders_INCLUDE_DIR
      NAMES vulkan/vk_layer.h
      HINTS ${VULKAN_HEADERS_INSTALL_DIR}/include
      NO_CMAKE_FIND_ROOT_PATH)
else()
  message(FATAL_ERROR "VULKAN_HEADERS_INSTALL_DIR must be specified!")
endif()

message(STATUS "VulkanHeaders ${VulkanHeaders_INCLUDE_DIR}")

if(DEFINED VULKAN_LOADER_GENERATED_DIR)
  find_path(VulkanLoaderGenerated_INCLUDE_DIR
      NAMES vk_layer_dispatch_table.h
      HINTS ${VULKAN_LOADER_GENERATED_DIR}
      NO_CMAKE_FIND_ROOT_PATH)
else()
  message(FATAL_ERROR "VULKAN_LOADER_GENERATED_DIR must be specified!")
endif()
message(STATUS "VulkanLoaderGenerated ${VulkanLoaderGenerated_INCLUDE_DIR}")

add_library(VkLayer_stadia_pipeline_compile_time SHARED
    layer/layer_data.cc
    layer/compile_time_layer.cc
)
target_include_directories(VkLayer_stadia_pipeline_compile_time PRIVATE
    layer
    ${VulkanHeaders_INCLUDE_DIR}
    ${VulkanLoaderGenerated_INCLUDE_DIR}
    third_party/farmhash/src
)
target_link_libraries(VkLayer_stadia_pipeline_compile_time PRIVATE
    absl::flat_hash_map
    absl::flat_hash_set
    absl::strings
    absl::str_format
    absl::synchronization
    farmhash
)
target_compile_definitions(VkLayer_stadia_pipeline_compile_time PRIVATE
	STADIA_PERFORMANCE_LAYERS_NO_GLOG=1
)

add_library(VkLayer_stadia_pipeline_runtime SHARED
    layer/layer_data.cc
    layer/runtime_layer.cc
    layer/runtime_layer_data.cc
)
target_include_directories(VkLayer_stadia_pipeline_runtime PRIVATE
    layer
    ${VulkanHeaders_INCLUDE_DIR}
    ${VulkanLoaderGenerated_INCLUDE_DIR}
    third_party/farmhash/src
)
target_link_libraries(VkLayer_stadia_pipeline_runtime PRIVATE
    absl::flat_hash_map
    absl::flat_hash_set
    absl::strings
    absl::str_format
    absl::synchronization
    farmhash
)
target_compile_definitions(VkLayer_stadia_pipeline_runtime PRIVATE
	STADIA_PERFORMANCE_LAYERS_NO_GLOG=1
)

add_library(VkLayer_stadia_frame_time SHARED
    layer/layer_data.cc
    layer/log_scanner.cc
    layer/frame_time_layer.cc
)
target_include_directories(VkLayer_stadia_frame_time PRIVATE
    layer
    ${VulkanHeaders_INCLUDE_DIR}
    ${VulkanLoaderGenerated_INCLUDE_DIR}
    third_party/farmhash/src
)
target_link_libraries(VkLayer_stadia_frame_time PRIVATE
    absl::flat_hash_map
    absl::flat_hash_set
    absl::strings
    absl::str_format
    absl::synchronization
    farmhash
)
target_compile_definitions(VkLayer_stadia_frame_time PRIVATE
	STADIA_PERFORMANCE_LAYERS_NO_GLOG=1
)

install(TARGETS
        VkLayer_stadia_pipeline_compile_time
        VkLayer_stadia_pipeline_runtime
        VkLayer_stadia_frame_time
        DESTINATION lib)

install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/layer
        DESTINATION ${CMAKE_INSTALL_PREFIX}
        FILES_MATCHING PATTERN "*.json")

enable_testing()
add_executable(layer_support_tests
    layer/layer_data.cc
    layer/log_scanner.cc
    units/log_scanner_tests.cc
)
target_include_directories(layer_support_tests PRIVATE
    layer
    ${VulkanHeaders_INCLUDE_DIR}
    ${VulkanLoaderGenerated_INCLUDE_DIR}
    third_party/farmhash/src
    third_party/googletest/googletest/include
)
target_link_libraries(layer_support_tests PRIVATE
    absl::flat_hash_map
    absl::flat_hash_set
    absl::strings
    absl::str_format
    absl::synchronization
    farmhash
    gtest
    gtest_main
    ${FILESYSTEM_LIB_NAME}
)
target_compile_definitions(layer_support_tests PRIVATE
	STADIA_PERFORMANCE_LAYERS_NO_GLOG=1
)
add_test(layer_support_tests layer_support_tests)
add_custom_target(check COMMAND layer_support_tests)
add_dependencies(check layer_support_tests)


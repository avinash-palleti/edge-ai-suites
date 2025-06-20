# ==============================================================================
# Copyright (C) 2018-2024 Intel Corporation
#
# SPDX-License-Identifier: MIT
# ==============================================================================

set (TARGET_NAME "image_inference_openvino")

# FIXME: update macro below
find_package(PkgConfig REQUIRED)
find_package(OpenVINO REQUIRED Runtime)

set(OpenVINO_MIN_VERSION "2024.0.0")
if(${OpenVINO_VERSION} VERSION_LESS ${OpenVINO_MIN_VERSION})
        message(FATAL_ERROR "OpenVINO version should be >= ${OpenVINO_MIN_VERSION}. Found: ${OpenVINO_VERSION}")
endif()

file (GLOB MAIN_SRC
        ${CMAKE_CURRENT_SOURCE_DIR}/*.cpp
        ${CMAKE_CURRENT_SOURCE_DIR}/*.c
        )

file (GLOB MAIN_HEADERS
        ${CMAKE_CURRENT_SOURCE_DIR}/*.hpp
        ${CMAKE_CURRENT_SOURCE_DIR}/*.h
        )
include_directories(${CMAKE_SOURCE_DIR}/ai_inference/include
${CMAKE_SOURCE_DIR}/ai_inference/inference_backend/include
${CMAKE_SOURCE_DIR}/ai_inference/3rdParty/dlstreamer/openvino/mappers
)
add_library(${TARGET_NAME} STATIC ${MAIN_SRC} ${MAIN_HEADERS})
set_compile_flags(${TARGET_NAME})

# FIXME: there are some debug information that are removed for release build type
# FIXME: hence it marked as error
if(UNIX)
    target_compile_options(${TARGET_NAME} PRIVATE -Wno-error=unused-parameter)
endif()

target_include_directories(${TARGET_NAME}
PUBLIC
        ${CMAKE_CURRENT_SOURCE_DIR}
        ${OpenVINO_INCLUDE_DIRS}
)

target_link_libraries(${TARGET_NAME}
PUBLIC
        openvino::runtime
        utils
        logger
        # dlstreamer_logger
)

if (${ENABLE_VAAPI} AND ${OpenVINO_VERSION} VERSION_GREATER_EQUAL "2023.3")
        add_definitions(-DENABLE_GPU_TILE_AFFINITY)
        target_link_libraries(${TARGET_NAME} PUBLIC va_api_wrapper)
endif()

cmake_minimum_required(VERSION 3.12)

set(CMAKE_C_STANDARD 11)
set(CMAKE_CXX_STANDARD 17)

set(TARGET_NAME FreeRTOS_PICO2350)

include(${PICO_SDK_PATH}/pico_sdk_init.cmake)

project(${TARGET_NAME}  C CXX ASM)
include(${CMAKE_CURRENT_SOURCE_DIR}/../src/CMakeLists.txt)




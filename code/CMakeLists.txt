cmake_minimum_required(VERSION 3.13)

set(CMAKE_C_STANDARD 11)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

# == DO NEVER EDIT THE NEXT LINES for Raspberry Pi Pico VS Code Extension to work ==
if(WIN32)
    set(USERHOME $ENV{USERPROFILE})
else()
    set(USERHOME $ENV{HOME})
endif()
set(sdkVersion 2.0.0)
set(toolchainVersion 13_2_Rel1)
set(picotoolVersion 2.0.0)
include(${USERHOME}/.pico-sdk/cmake/pico-vscode.cmake)
# ====================================================================================
set(PICO_BOARD pico2 CACHE STRING "Board type")

set(TARGET_NAME FreeRTOS_PICO2350)

include(pico_sdk_import.cmake)

project(FreeRTOS_PICO2350 C CXX ASM)

include(${CMAKE_CURRENT_SOURCE_DIR}/../src/CMakeLists.txt)
pico_set_program_name(${TARGET_NAME} STRING(${TARGET_NAME}))
pico_set_program_version(${TARGET_NAME} "1.0")
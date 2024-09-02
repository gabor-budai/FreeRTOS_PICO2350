cmake_minimum_required(VERSION 3.12)

set(SRC ${CMAKE_CURRENT_SOURCE_DIR}/../src)
set(FREERTOS_KERNEL_PATH ${SRC}/FreeRTOS)

pico_sdk_init()

add_subdirectory(${FREERTOS_KERNEL_PATH}/portable/ThirdParty/GCC/RP2350_ARM_NTZ FREERTOS_KERNEL)

add_executable(${TARGET_NAME} 
    ${SRC}/hello_freertos.c 
)

target_link_libraries(${TARGET_NAME}
    pico_async_context_freertos
    FreeRTOS-Kernel-Heap4
    pico_stdlib)

target_include_directories(${TARGET_NAME} PRIVATE
    ${SRC}
)

pico_add_extra_outputs(${TARGET_NAME})    
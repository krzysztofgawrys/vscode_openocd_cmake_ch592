cmake_minimum_required(VERSION 3.16)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

set(CMAKE_SYSTEM_NAME Generic)
set(CMAKE_SYSTEM_PROCESSOR riscv)

set(CMAKE_TRY_COMPILE_TARGET_TYPE STATIC_LIBRARY)

project(SoilSensor_CH592 C ASM)

set(CPU_FLAGS "-march=rv32imac_zicsr_zifencei -mabi=ilp32 -mcmodel=medany -msmall-data-limit=8 -mno-save-restore")

set(CMAKE_C_STANDARD 99)
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${CPU_FLAGS} -ffunction-sections -fdata-sections -Og -g3")
set(CMAKE_ASM_FLAGS "${CMAKE_ASM_FLAGS} ${CPU_FLAGS}")

add_definitions(-DDEBUG -DCH592)

file(GLOB_RECURSE SOURCES 
    "src/*.c"
    "StdPeriphDriver/*.c"
    "Startup/startup_CH592.S" 
)

include_directories(
    "src"
    "StdPeriphDriver/inc"
    "RVMSIS"
)

add_executable(${PROJECT_NAME}.elf ${SOURCES})

target_link_libraries(${PROJECT_NAME}.elf 
    PRIVATE 
    ${CMAKE_SOURCE_DIR}/LIB/libCH59xBLE.a
    ${CMAKE_SOURCE_DIR}/StdPeriphDriver/libISP592.a
)

set(LINKER_SCRIPT "${CMAKE_SOURCE_DIR}/Link.ld")
set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} ${CPU_FLAGS} -T${LINKER_SCRIPT} -nostartfiles -Xlinker --gc-sections --specs=nano.specs --specs=nosys.specs")

add_custom_command(TARGET ${PROJECT_NAME}.elf POST_BUILD
    COMMAND riscv-wch-elf-objcopy -O ihex ${PROJECT_NAME}.elf ${PROJECT_NAME}.hex
    COMMAND riscv-wch-elf-objcopy -O binary ${PROJECT_NAME}.elf ${PROJECT_NAME}.bin
    COMMAND riscv-wch-elf-size ${PROJECT_NAME}.elf
    COMMENT "Building HEX and BIN files..."
)

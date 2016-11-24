cmake_minimum_required(VERSION 3.4)

# Options
option(CMAKE_BUILD_TYPE "" RELEASE)
option(TARGET   "" NONE)
option(FORKNAME "" cleanflight)

# Toolchain setup #####
set(CMAKE_SYSTEM_NAME Generic)
set(CMAKE_SYSTEM_PROCESSOR arm)

set(arch arm-none-eabi)
set(CMAKE_C_COMPILER   ${arch}-gcc)
set(CMAKE_CXX_COMPILER ${arch}-g++)
set(CMAKE_OBJCOPY      ${arch}-objcopy)
set(CMAKE_SIZE         ${arch}-size)

# Target Flags

set(CMAKE_ASM_COMPILE_FLAGS_RELEASE "")
set(CMAKE_C_COMPILE_FLAGS_RELEASE "")
set(CMAKE_CXX_COMPILE_FLAGS_RELEASE "")
set(CMAKE_EXE_LINKER_FLAGS_RELEASE "-lm -lc -lnosys -nostartfiles" )

set(CMAKE_ASM_COMPILE_FLAGS_DEBUG "")
set(CMAKE_C_COMPILE_FLAGS_DEBUG "")
set(CMAKE_CXX_COMPILE_FLAGS_DEBUG "")
set(CMAKE_EXE_LINKER_FLAGS_DEBUG "-lm -lc -lnosys -nostartfiles" )

# Enable ccache
find_program(CCACHE_PROGRAM ccache)
if(CCACHE_PROGRAM)
    set_property(GLOBAL PROPERTY RULE_LAUNCH_COMPILE "${CCACHE_PROGRAM}")
endif()

enable_testing()

project(
  ${FORKNAME}_${TARGET}
  VERSION 0.0.1 
  LANGUAGES C ASM
)

#configure_file(
#  ${CMAKE_CURRENT_SOURCE_DIR}/version.h.in
#  ${CMAKE_CURRENT_BINARY_DIR}/version.h
#)

include_directories(
  ${CMAKE_CURRENT_BINARY_DIR}
  ${CMAKE_CURRENT_SOURCE_DIR}
)

set( SOURCES
  main.c
)

set( LIB_SOURCES

)

add_executable(${PROJECT_NAME}.elf ${SOURCES} ${LIB_SOURCES})

add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD COMMAND ${CMAKE_SIZE} ${PROJECT_NAME})

add_custom_target(${PROJECT_NAME}.hex
  DEPENDS ${PROJECT_NAME} 
  COMMAND ${CMAKE_OBJCOPY} -Oihex ${PROJECT_NAME}.elf --set-start 0x800000 ${PROJECT_NAME}.hex
)
add_custom_target(${PROJECT_NAME}.bin
  DEPENDS ${PROJECT_NAME} 
  COMMAND ${CMAKE_OBJCOPY} -Obinary ${PROJECT_NAME}.elf ${PROJECT_NAME}.bin
)

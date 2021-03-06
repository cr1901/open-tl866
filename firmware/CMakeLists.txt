cmake_minimum_required(VERSION 3.5)

# set up the Microchip cross toolchain
set(CMAKE_TOOLCHAIN_FILE external/cmake-microchip/toolchain.cmake)

# set the default MCU model
set(MICROCHIP_MCU PIC18F87J50)


project(open-tl866 C)

set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/dist)


add_library(core INTERFACE)

target_sources(core INTERFACE
    ${CMAKE_SOURCE_DIR}/main.c

    ${CMAKE_SOURCE_DIR}/comlib.c
    ${CMAKE_SOURCE_DIR}/configuration_bits.c
    ${CMAKE_SOURCE_DIR}/io.c
    ${CMAKE_SOURCE_DIR}/parse.c
    ${CMAKE_SOURCE_DIR}/stock_compat.c
    ${CMAKE_SOURCE_DIR}/usb/usb_descriptors.c
)

# set linker options to support the bootloader
target_link_libraries(core INTERFACE
    # reserve the bootloader code area
    "--codeoffset=1800"
    # reserve the bootloader data area
    "--rom=default,-1fc00-1ffff"
    # set the firmware magic number
    "--serial=55aaa55a@1fbfc"
)

# include M-Stack USB stack with CDC ACM serial
find_package(MStack)
target_link_libraries(core INTERFACE MStack_CDC)
target_include_directories(core
    # lets MStack find usb_config.h
    INTERFACE ${CMAKE_SOURCE_DIR}/usb
)


function(add_tl866_mode name)
    add_executable(${name} ${ARGN})

    target_link_libraries(${name} PRIVATE
        # include the rest of the firmware
        core

        # select linker output files
        "--output=intel,elf"
        "--asmlist"
        "-m" # memory map
    )
endfunction()


add_tl866_mode(tl866-bitbang
    ${CMAKE_SOURCE_DIR}/modes/bitbang/bitbang.c
)

add_tl866_mode(tl866-at89
    ${CMAKE_SOURCE_DIR}/modes/programmer/at89/at89.c
)

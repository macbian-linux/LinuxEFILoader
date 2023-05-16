# thank you https://gist.github.com/tomtzook/6c93dfe4a3eb8f17aae04322a95f9c61

cmake_minimum_required(VERSION 3.16)
project(LinuxEFILoader C)

if (NOT CMAKE_RUNTIME_OUTPUT_DIRECTORY)
    set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}")
endif()
if (NOT CMAKE_LIBRARY_OUTPUT_DIRECTORY)
    set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/lib")
endif()

set(EFI_INCLUDE_PATH "/usr/include/efi")
set(EFI_LDS_PATH "${EFI_LIBRARY_DIR}/elf_${ARCH}_efi.lds")
set(EFI_LIBRARY_DIR "/usr/lib")
set(EFI_CRT_OBJS "${EFI_LIBRARY_DIR}/crt0-efi-${ARCH}.o")

set(ARCH x86_64)
set(CMAKE_C_STANDARD 11)
set(CMAKE_C_COMPILER /usr/bin/gcc)
set(CMAKE_LINKER /usr/bin/ld)
set(CMAKE_OBJCOPY /usr/bin/objcopy)
set(CMAKE_C_CREATE_SHARED_LIBRARY "<CMAKE_LINKER> <CMAKE_C_LINK_FLAGS> <LINK_FLAGS> <OBJECTS> ${EFI_LIBRARY_DIR}/crt0-efi-${ARCH}.o -o <TARGET> <LINK_LIBRARIES>")

list(APPEND COMPILE_FLAGS "-fpic" "-g" "-ffreestanding" "-fno-stack-protector" "-fno-stack-check" "-fshort-wchar" "-mno-red-zone" "-maccumulate-outgoing-args" "-Wall" "-DEFI_FUNCTION_WRAPPER")
set(LINK_FLAGS -shared -Bsymbolic -L${EFI_LIBRARY_DIR} -T${EFI_LIBRARY_DIR}/elf_${ARCH}_efi.lds)

set(TARGET_NAME LinuxEFILoader)

include_directories(${EFI_INCLUDE_PATH})

add_library(${TARGET_NAME} SHARED main.c)
target_compile_options(${TARGET_NAME} PRIVATE ${COMPILE_FLAGS})
target_link_options(${TARGET_NAME} PRIVATE ${LINK_FLAGS})
target_link_libraries(${TARGET_NAME} PRIVATE efi gnuefi)

add_custom_command(TARGET "${TARGET_NAME}" POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E make_directory ${CMAKE_RUNTIME_OUTPUT_DIRECTORY})


function(efi_objcopy)
    cmake_parse_arguments(parsedargs "" "INFILE;OUTFILE" "SECTIONS" ${ARGN})
    add_custom_command(TARGET "${TARGET_NAME}" POST_BUILD
            COMMAND ${CMAKE_OBJCOPY}
            ${parsedargs_SECTIONS}
            --target=efi-app-x86_64 #  "efi-app", "efi-bsd", "efi-rtd"
            --subsystem=10
            "${parsedargs_INFILE}"
            "${parsedargs_OUTFILE}")
endfunction()

set(ELF_SECTIONS
        -j .text
        -j .sdata
        -j .data
        -j .dynamic
        -j .dynsym
        -j .rel
        -j .rela
        -j .rel.*
        -j .rela.*
        -j .reloc)

efi_objcopy(INFILE "${CMAKE_LIBRARY_OUTPUT_DIRECTORY}/lib${TARGET_NAME}.so"
        OUTFILE "${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/${TARGET_NAME}.efi"
        SECTIONS ${ELF_SECTIONS})
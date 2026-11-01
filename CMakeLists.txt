cmake_minimum_required(VERSION 3.22)

project(dpiawaremouse
    VERSION 0.1.0
    DESCRIPTION "DPI-aware mouse crossing tool"
    HOMEPAGE_URL "https://github.com/simonbeaudoin0935/DPIAwareMouse"
    LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

include(GNUInstallDirs)
include(CheckCXXCompilerFlag)
include(CheckCXXSourceCompiles)
include(CheckLinkerFlag)
include(CheckIPOSupported)
include(CheckPIESupported)

if(NOT CMAKE_CONFIGURATION_TYPES AND NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE RelWithDebInfo CACHE STRING "Build type" FORCE)
endif()

if(NOT CMAKE_CXX_COMPILER_ID MATCHES "^(GNU|Clang)$")
    message(FATAL_ERROR "The compiler policy currently requires GCC or Clang.")
endif()

add_executable(dpiawaremouse
    src/main.cpp)

target_compile_definitions(dpiawaremouse PRIVATE
    DPIAWAREMOUSE_VERSION="${PROJECT_VERSION}"
    _GLIBCXX_ASSERTIONS)

target_compile_options(dpiawaremouse PRIVATE
    -Wall -Wextra -Wpedantic -Werror
    -Wformat=2 -Wformat-security -Wconversion -Wsign-conversion
    -Wshadow -Wcast-qual -Wold-style-cast -Woverloaded-virtual
    -Wnon-virtual-dtor -Wnull-dereference -Wdouble-promotion
    -Wimplicit-fallthrough -Wvla -Wundef -Wswitch-enum
    -Wmissing-declarations -Wredundant-decls -Wwrite-strings -Walloca
    -fstack-protector-all -fno-omit-frame-pointer
    "$<$<CONFIG:Release,RelWithDebInfo>:-O3>")

if(CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
    target_compile_options(dpiawaremouse PRIVATE
        -Wcast-align=strict -Wformat-overflow=2 -Wformat-truncation=2
        -Wduplicated-cond -Wduplicated-branches -Wlogical-op
        -Wuseless-cast -Warray-bounds=2)
else()
    target_compile_options(dpiawaremouse PRIVATE -Wcast-align)
endif()

foreach(flag IN ITEMS
        -fstack-clash-protection
        -fcf-protection=full
        -ftrivial-auto-var-init=zero)
    string(MAKE_C_IDENTIFIER "HAVE_${flag}" capability)
    check_cxx_compiler_flag("${flag}" "${capability}")
    if(${capability})
        target_compile_options(dpiawaremouse PRIVATE "${flag}")
    else()
        message(WARNING "Compiler/target does not support ${flag}; protection unavailable.")
    endif()
endforeach()

# Fortification needs optimization and libc support, not just flag acceptance.
set(CMAKE_REQUIRED_FLAGS "-O2 -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=3")
check_cxx_source_compiles("
    #include <features.h>
    #if !defined(__USE_FORTIFY_LEVEL) || __USE_FORTIFY_LEVEL < 3
    #error Fortify level 3 is unavailable
    #endif
    int main() { return 0; }
" HAVE_FORTIFY_3)
if(HAVE_FORTIFY_3)
    set(fortify_level 3)
else()
    set(CMAKE_REQUIRED_FLAGS "-O2 -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=2")
    check_cxx_source_compiles("
        #include <features.h>
        #if !defined(__USE_FORTIFY_LEVEL) || __USE_FORTIFY_LEVEL < 2
        #error Fortify level 2 is unavailable
        #endif
        int main() { return 0; }
    " HAVE_FORTIFY_2)
    if(HAVE_FORTIFY_2)
        set(fortify_level 2)
        message(WARNING "Fortify level 3 unavailable; using level 2.")
    else()
        message(FATAL_ERROR "The compiler/libc must support _FORTIFY_SOURCE >= 2.")
    endif()
endif()
unset(CMAKE_REQUIRED_FLAGS)
target_compile_options(dpiawaremouse PRIVATE
    "$<$<CONFIG:Release,RelWithDebInfo,MinSizeRel>:-U_FORTIFY_SOURCE>"
    "$<$<CONFIG:Release,RelWithDebInfo,MinSizeRel>:-D_FORTIFY_SOURCE=${fortify_level}>")

check_pie_supported(OUTPUT_VARIABLE pie_error LANGUAGES CXX)
if(NOT CMAKE_CXX_LINK_PIE_SUPPORTED)
    message(FATAL_ERROR "PIE is required: ${pie_error}")
endif()
set_property(TARGET dpiawaremouse PROPERTY POSITION_INDEPENDENT_CODE TRUE)

foreach(flag IN ITEMS
        -Wl,-z,relro -Wl,-z,now -Wl,-z,noexecstack -Wl,--fatal-warnings)
    string(MAKE_C_IDENTIFIER "HAVE_${flag}" capability)
    check_linker_flag(CXX "${flag}" "${capability}")
    if(NOT ${capability})
        message(FATAL_ERROR "Required linker protection is unavailable: ${flag}")
    endif()
    target_link_options(dpiawaremouse PRIVATE "${flag}")
endforeach()

check_ipo_supported(RESULT ipo_supported OUTPUT ipo_error LANGUAGES CXX)
if(NOT ipo_supported)
    message(FATAL_ERROR "Release builds require IPO/LTO support: ${ipo_error}")
endif()
set_target_properties(dpiawaremouse PROPERTIES
    INTERPROCEDURAL_OPTIMIZATION_RELEASE TRUE
    INTERPROCEDURAL_OPTIMIZATION_RELWITHDEBINFO TRUE
    INTERPROCEDURAL_OPTIMIZATION_MINSIZEREL TRUE)
target_link_options(dpiawaremouse PRIVATE
    "$<$<CONFIG:Release,RelWithDebInfo>:-O3>")

install(TARGETS dpiawaremouse
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})

install(FILES man/dpiawaremouse.1
    DESTINATION ${CMAKE_INSTALL_MANDIR}/man1)

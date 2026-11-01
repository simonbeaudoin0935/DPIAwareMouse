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

add_executable(dpiawaremouse
    src/main.cpp)

target_compile_definitions(dpiawaremouse PRIVATE
    DPIAWAREMOUSE_VERSION="${PROJECT_VERSION}")

install(TARGETS dpiawaremouse
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})

install(FILES man/dpiawaremouse.1
    DESTINATION ${CMAKE_INSTALL_MANDIR}/man1)

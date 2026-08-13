cmake_minimum_required(VERSION 3.20)
project(plasma-notification-mirror LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

find_package(Qt6 REQUIRED COMPONENTS Core Gui)
find_package(LayerShellQt REQUIRED)
find_package(PkgConfig REQUIRED)
pkg_check_modules(LIBSYSTEMD REQUIRED IMPORTED_TARGET libsystemd)

add_executable(plasma-notification-mirror
    src/main.cpp
)

target_link_libraries(plasma-notification-mirror
    PRIVATE
        Qt6::Core
        Qt6::Gui
        LayerShellQt::Interface
        PkgConfig::LIBSYSTEMD
)

include(GNUInstallDirs)

install(TARGETS plasma-notification-mirror
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
)

install(FILES config/config.example.ini
    DESTINATION ${CMAKE_INSTALL_DATADIR}/plasma-notification-mirror
)

install(FILES systemd/plasma-notification-mirror.service
    DESTINATION ${CMAKE_INSTALL_DATADIR}/systemd/user
)

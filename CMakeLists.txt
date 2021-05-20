cmake_minimum_required(VERSION 3.12)

project(libexample
        LANGUAGES C
        VERSION "1.0.0")

set(CMAKE_C_STANDARD 90)
set(CMAKE_C_STANDARD_REQUIRED ON)
set(CMAKE_C_EXTENSIONS OFF)

option(BUILD_SHARED_LIBS "Whether to build a dynamic library" OFF)

# library can be either static or shared, controlled by BUILD_SHARED_LIBS during configuration
add_library(${PROJECT_NAME} src/library.c)
# this target allows subdirectory builds to have fake imported target which normally imports with find_package
add_library(EXAMPLE::Libexample ALIAS ${PROJECT_NAME})

include(GenerateExportHeader)
# generate libexample_import.h with needed platform-dependent way to export symbols in a shared lib
generate_export_header(${PROJECT_NAME})

# for CMAKE_INSTALL_* variables
include(GNUInstallDirs)

target_include_directories(${PROJECT_NAME}
        PUBLIC
        # will be used when imported using config file as find_package
        $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
        # will be used during subdirectory build
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
        $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}>)


# declare headers to be installed
set_property(TARGET ${PROJECT_NAME}
        PROPERTY PUBLIC_HEADER
        include/library.h
        "${CMAKE_CURRENT_BINARY_DIR}/libexample_export.h")
# export libexample target as Libexample inside LibexampleConfig.cmake file when install(TARGETS) is invoked
set_target_properties(${PROJECT_NAME}
        PROPERTIES
        EXPORT_NAME Libexample
        PREFIX "")

# install libexample target files and create and EXPORT we will use later on
install(TARGETS ${PROJECT_NAME}
        EXPORT LibexampleConfig

        # you can omit all these lines if your project is cmake 3.14+ and GNUInstallDirs was already included
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
        PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
        )

# installs LibexampleConfig.cmake and LibexampleConfig-[build-type-here].cmake into
# ${CMAKE_INSTALL_PREFIX}/${CMAKE_INSTALL_DATADIR}/cmake/
# EXPORT is provided by install(TARGETS)
install(EXPORT LibexampleConfig
        NAMESPACE EXAMPLE::
        DESTINATION "${CMAKE_INSTALL_DATADIR}/cmake"
        )

# creates LibexampleConfig.cmake in binary dir during configuration
export(TARGETS ${PROJECT_NAME}
        NAMESPACE EXAMPLE::
        FILE "${CMAKE_CURRENT_BINARY_DIR}/LibexampleConfig.cmake"
        )

cmake_minimum_required(VERSION 3.1)
cmake_policy(SET CMP0063 NEW)

project(flecs-dash-amalgamated LANGUAGES C)

option(FLECS_DASH_STATIC_LIBS "Build static flecs lib" ON)
option(FLECS_DASH_PIC "Compile static flecs lib with position-independent-code (PIC)" ON)
option(FLECS_DASH_SHARED_LIBS "Build shared flecs lib" ON)
option(FLECS_DASH_DEVELOPER_WARNINGS "Enable more warnings" OFF)

if(NOT FLECS_DASH_STATIC_LIBS AND NOT FLECS_DASH_SHARED_LIBS)
    message(FATAL_ERROR "At least one of FLECS_DASH_STATIC_LIBS or FLECS_DASH_SHARED_LIBS options must be enabled")
endif()

# include utilities for compiler options and warnings

include(cmake/target_default_compile_warnings.cmake)
include(cmake/target_default_compile_options.cmake)

# collect all include and source files
# (use the manual list to improve ide support)

set(INC
        include/flecs_components_http.h
        include/flecs_dash.h
        include/flecs_json.h
        include/flecs_meta.h
        include/flecs_monitor.h
        include/flecs_player.h
        include/flecs_rest.h
        include/flecs_systems_civetweb.h)

set(SRC
        src/flecs_components_http.c
        src/flecs_dash.c
        src/flecs_json.c
        src/flecs_meta.c
        src/flecs_monitor.c
        src/flecs_player.c
        src/flecs_rest.c
        src/flecs_systems_civetweb.c)

set(FLECS_DASH_TARGETS "")

# build the shared library
if(FLECS_DASH_SHARED_LIBS)
    add_library(flecs-dash-amalgamated SHARED ${INC} ${SRC})
    add_library(flecs::flecs-dash-amalgamated ALIAS flecs-dash-amalgamated)
    target_default_compile_options_c(flecs-dash-amalgamated)
    if(FLECS_DASH_DEVELOPER_WARNINGS)
        target_default_compile_warnings_c(flecs-dash-amalgamated)
    endif()

    target_include_directories(flecs-dash-amalgamated PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
        $<INSTALL_INTERFACE:include>)
    target_link_libraries (flecs-dash-amalgamated flecs)

    list(APPEND FLECS_DASH_TARGETS flecs-dash-amalgamated)
endif()

# build the static library
if(FLECS_DASH_STATIC_LIBS)
    add_library(flecs-dash-amalgamated_static STATIC ${INC} ${SRC})
    add_library(flecs::flecs-dash-amalgamated_static ALIAS flecs-dash-amalgamated_static)
    target_default_compile_options_c(flecs-dash-amalgamated_static)
    if(FLECS_DASH_DEVELOPER_WARNINGS)
        target_default_compile_warnings_c(flecs-dash-amalgamated_static)
    endif()

    if(FLECS_DASH_PIC)
        set_property(TARGET flecs-dash-amalgamated_static PROPERTY POSITION_INDEPENDENT_CODE ON)
    endif()

    target_include_directories(flecs-dash-amalgamated_static PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
        $<INSTALL_INTERFACE:include>)
    target_link_libraries (flecs-dash-amalgamated_static flecs)

    list(APPEND FLECS_DASH_TARGETS flecs-dash-amalgamated_static)
endif()


# define the install steps
include(GNUInstallDirs)
install(DIRECTORY "${PROJECT_SOURCE_DIR}/include/"
        DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
        FILES_MATCHING
        PATTERN "*.h"
        PATTERN "*.hpp")

install(TARGETS ${FLECS_DASH_TARGETS}
        EXPORT flecs-export
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR})

install(EXPORT flecs-export
        DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/flecs-dash-amalgamated
        NAMESPACE flecs::
        FILE flecs-config.cmake)

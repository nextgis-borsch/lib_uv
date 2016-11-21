################################################################################
# Project:  Lib JBIG
# Purpose:  CMake build scripts
# Author:   Dmitry Baryshnikov, dmitry.baryshnikov@nexgis.com
# Author:   NikitaFeodonit, nfeodonit@yandex.com
################################################################################
# Copyright (C) 2015, NextGIS <info@nextgis.com>
#
# Permission is hereby granted, free of charge, to any person obtaining a
# copy of this software and associated documentation files (the "Software"),
# to deal in the Software without restriction, including without limitation
# the rights to use, copy, modify, merge, publish, distribute, sublicense,
# and/or sell copies of the Software, and to permit persons to whom the
# Software is furnished to do so, subject to the following conditions:
#
# The above copyright notice and this permission notice shall be included
# in all copies or substantial portions of the Software.
#
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS
# OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL
# THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING
# FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER
# DEALINGS IN THE SOFTWARE.
################################################################################

cmake_minimum_required(VERSION 2.8.12 FATAL_ERROR)

project(libuv C)

# some init settings
set(CMAKE_COLOR_MAKEFILE ON)
# set path to additional CMake modules
set(CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/cmake ${CMAKE_MODULE_PATH})

include(GNUInstallDirs)

set(INSTALL_BIN_DIR ${CMAKE_INSTALL_BINDIR} CACHE INTERNAL "Installation directory for executables" FORCE)
set(INSTALL_LIB_DIR ${CMAKE_INSTALL_LIBDIR} CACHE INTERNAL "Installation directory for libraries" FORCE)
set(INSTALL_INC_DIR ${CMAKE_INSTALL_INCLUDEDIR} CACHE INTERNAL "Installation directory for headers" FORCE)
set(INSTALL_DOC_DIR ${CMAKE_INSTALL_DOCDIR} CACHE INTERNAL "Share directory for data" FORCE)

set(INSTALL_INC_FULL_DIR "${CMAKE_INSTALL_PREFIX}/${INSTALL_INC_DIR}")

if(CMAKE_GENERATOR_TOOLSET MATCHES "v([0-9]+)_xp")
    add_definitions(-D_WIN32_WINNT=0x0501)
endif()

include(util)
check_version(UV_MAJOR_VERSION UV_MINOR_VERSION UV_REV_VERSION)
set(VERSION ${UV_MAJOR_VERSION}.${UV_MINOR_VERSION}.${UV_REV_VERSION})
report_version(${PROJECT_NAME} ${VERSION})

configure_file(${CMAKE_SOURCE_DIR}/cmake/cmake_uninstall.cmake.in ${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake IMMEDIATE @ONLY)


if(${CMAKE_SYSTEM_NAME} STREQUAL "Linux")
    set(LINUX TRUE)
endif()


set(PUBLIC_HEADER_DIR "${CMAKE_SOURCE_DIR}/include")

set(PUBLIC_HDRS
    include/uv.h
    include/uv-errno.h
    include/uv-threadpool.h
    include/uv-version.h
)

set(SRCS
    src/fs-poll.c
    src/heap-inl.h
    src/inet.c
    src/queue.h
    src/threadpool.c
    src/uv-common.c
    src/uv-common.h
    src/version.c
)

set(DOCS
    LICENSE
)

include_directories ( ${CMAKE_SOURCE_DIR}/include )
include_directories ( ${CMAKE_SOURCE_DIR}/src )


if (WIN32)
    list(APPEND PUBLIC_HDRS
        include/uv-win.h
        include/tree.h
    )
    list(APPEND SRCS
        src/win/async.c
        src/win/atomicops-inl.h
        src/win/core.c
        src/win/dl.c
        src/win/error.c
        src/win/fs-event.c
        src/win/fs.c
        src/win/getaddrinfo.c
        src/win/getnameinfo.c
        src/win/handle.c
        src/win/handle-inl.h
        src/win/internal.h
        src/win/loop-watcher.c
        src/win/pipe.c
        src/win/poll.c
        src/win/process-stdio.c
        src/win/process.c
        src/win/req.c
        src/win/req-inl.h
        src/win/signal.c
        src/win/stream.c
        src/win/stream-inl.h
        src/win/tcp.c
        src/win/thread.c
        src/win/timer.c
        src/win/tty.c
        src/win/udp.c
        src/win/util.c
        src/win/winapi.c
        src/win/winapi.h
        src/win/winsock.c
        src/win/winsock.h
    )
    include_directories ( ${CMAKE_SOURCE_DIR}/src/win )
    set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DWIN32_LEAN_AND_MEAN -D_WIN32_WINNT=0x0600")
# TODO: LIBS += -lws2_32 -lpsapi -liphlpapi -lshell32 -luserenv
endif()

if (UNIX)
    list(APPEND PUBLIC_HDRS
        include/uv-unix.h
    )
    list(APPEND SRCS
        src/unix/async.c
        src/unix/atomic-ops.h
        src/unix/core.c
        src/unix/dl.c
        src/unix/fs.c
        src/unix/getaddrinfo.c
        src/unix/getnameinfo.c
        src/unix/internal.h
        src/unix/loop-watcher.c
        src/unix/loop.c
        src/unix/pipe.c
        src/unix/poll.c
        src/unix/process.c
        src/unix/signal.c
        src/unix/spinlock.h
        src/unix/stream.c
        src/unix/tcp.c
        src/unix/thread.c
        src/unix/timer.c
        src/unix/tty.c
        src/unix/udp.c
    )
    include_directories ( ${CMAKE_SOURCE_DIR}/src/unix )
endif()

if (LINUX)
    list(APPEND PUBLIC_HDRS
        include/uv-linux.h
    )
    list(APPEND SRCS
        src/unix/linux-core.c
        src/unix/linux-inotify.c
        src/unix/linux-syscalls.c
        src/unix/linux-syscalls.h
        src/unix/proctitle.c
    )
    set (CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -D_GNU_SOURCE")
endif()

if (ANDROID)
    list(APPEND PUBLIC_HDRS
        include/android-ifaddrs.h
        include/pthread-fixes.h
    )
    list(APPEND SRCS
        src/unix/android-ifaddrs.c
        src/unix/pthread-fixes.c
    )
endif()

set(LIB_NAME ${PROJECT_NAME})
if(BUILD_SHARED_LIBS)
    add_library(${LIB_NAME} SHARED ${SRCS})
    set_target_properties(${LIB_NAME} PROPERTIES SOVERSION 1)
else()
    add_library(${LIB_NAME} STATIC ${SRCS})
    set_target_properties(${LIB_NAME} PROPERTIES SOVERSION 1)
endif()

set_property(TARGET ${LIB_NAME} PROPERTY C_STANDARD 90)
set_property(TARGET ${LIB_NAME} PROPERTY C_STANDARD_REQUIRED ON)
target_include_directories(${LIB_NAME} PUBLIC "${INSTALL_INC_FULL_DIR}")
target_sources(${LIB_NAME} PUBLIC ${PUBLIC_HDRS})

export(TARGETS ${LIB_NAME} FILE ${LIB_NAME}-exports.cmake EXPORT_LINK_INTERFACE_LIBRARIES)

add_custom_target(uninstall COMMAND ${CMAKE_COMMAND} -P ${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake)


if(NOT CYGWIN)
    set_target_properties(${LIB_NAME} PROPERTIES VERSION ${VERSION})
endif()

if(UNIX)
    # On unix-like platforms the library is almost always called lib${PROJECT_NAME}
#    set_target_properties(${LIB_NAME} PROPERTIES OUTPUT_NAME ${PROJECT_NAME})
    set_target_properties(${LIB_NAME} PROPERTIES OUTPUT_NAME uv)
endif()

if(NOT SKIP_INSTALL_LIBRARIES AND NOT SKIP_INSTALL_ALL)
    install(TARGETS ${LIB_NAME}
        EXPORT ${LIB_NAME}
        RUNTIME DESTINATION "${INSTALL_BIN_DIR}"
        ARCHIVE DESTINATION "${INSTALL_LIB_DIR}"
        LIBRARY DESTINATION "${INSTALL_LIB_DIR}" )
endif()

if(NOT SKIP_INSTALL_HEADERS AND NOT SKIP_INSTALL_ALL)
    install(DIRECTORY ${PUBLIC_HEADER_DIR}/ DESTINATION "${INSTALL_INC_DIR}")
    install(FILES ${DOCS} DESTINATION "${INSTALL_DOC_DIR}/lib${PROJECT_NAME}")
endif()

#
# CMakeLists.txt --- a "CMake" file for building LZO
#
# This file is part of the LZO data compression library.
#   http://www.oberhumer.com/opensource/lzo/
#
# Copyright (C) 1996-2017 Markus Franz Xaver Johannes Oberhumer
# All Rights Reserved.
#

cmake_minimum_required(VERSION 3.0 FATAL_ERROR)

#
# simple usage example (Unix):
#     mkdir -p build && cd build && cmake .. && make
#
# simple usage example (Windows MSVC):
#     md build
#     cd build
#     cmake .. -G "NMake Makefiles"
#     nmake
#
# another usage example (Unix):
#     mkdir -p build/release-i686
#     cd       build/release-i686
#     cmake ../.. -DENABLE_STATIC=0 -DENABLE_SHARED=1 \
#         -DCMAKE_C_COMPILER=gcc -DCMAKE_C_FLAGS="-m32 -march=i686" \
#         -DCMAKE_INSTALL_PREFIX=/opt/local/prefix-i686
#     make VERBOSE=1
#     make install
#
# see http://www.cmake.org/ for more info
#

# /***********************************************************************
# // init
# ************************************************************************/

project(lzo VERSION 2.10 LANGUAGES C)

# configuration options
if(BUILD_SHARED_LIBS AND WIN32)
    cmake_minimum_required(VERSION 3.4.3 FATAL_ERROR) # needed for WINDOWS_EXPORT_ALL_SYMBOLS
endif()

include(GNUInstallDirs)

# /***********************************************************************
# // targets
# ************************************************************************/

file(GLOB lzo_SOURCES "${CMAKE_CURRENT_SOURCE_DIR}/src/*.c")
list(SORT lzo_SOURCES)

# LZO library
add_library(lzo2 ${lzo_SOURCES})
set_target_properties(lzo2
  PROPERTIES
  OUTPUT_NAME lzo2
  OUTPUT_NAME_DEBUG lzo2d
)
if(BUILD_SHARED_LIBS)
    set_target_properties(lzo2 PROPERTIES SOVERSION 2 VERSION 2.0.0) # ABI version is 2.0.0 !
    set_target_properties(lzo2 PROPERTIES WINDOWS_EXPORT_ALL_SYMBOLS ON)
endif()

# /***********************************************************************
# // compilation flags
# // this section currently mostly matches the Autoconf version
# ************************************************************************/

include_directories("${CMAKE_CURRENT_SOURCE_DIR}/include")
include_directories("${CMAKE_CURRENT_SOURCE_DIR}")

include(CheckFunctionExists)
include(CheckIncludeFile)
include(CheckLibraryExists)
include(CheckSymbolExists)
include(CheckTypeSize)

if(MSVC)
    # disable silly warnings about using "deprecated" POSIX functions like fopen()
    add_definitions(-D_CRT_NONSTDC_NO_DEPRECATE)
    add_definitions(-D_CRT_NONSTDC_NO_WARNINGS)
    add_definitions(-D_CRT_SECURE_NO_DEPRECATE)
    add_definitions(-D_CRT_SECURE_NO_WARNINGS)
endif()

# Checks for typedefs and structures
macro(mfx_check_type_size type var)
    check_type_size("${type}" "mfx_${var}")
    if("${mfx_${var}}" MATCHES "^[1-9][0-9]*$")
        add_definitions(-D${var}=${mfx_${var}})
    else()
        set(mfx_${var} 0)
    endif()
endmacro()
# mfx_ACC_CHECK_SIZEOF  mfx_CHECK_SIZEOF
set(l short int long "long long" __int16 __int32 __int64 "void *"
size_t ptrdiff_t)
foreach(f ${l})
    string(TOUPPER "${f}" var)
    string(REGEX REPLACE " \\*" "_P" var "${var}")
    string(REGEX REPLACE "[^0-9A-Z_]" "_" var "${var}")
    mfx_check_type_size("${f}" "SIZEOF_${var}")
endforeach()

# LZO_HAVE_CONFIG_H
add_definitions(-DLZO_CFG_NO_CONFIG_HEADER=1)

# /***********************************************************************
# // "make install"
# ************************************************************************/

install(DIRECTORY include/lzo
    DESTINATION include
)

include(CMakePackageConfigHelpers)
write_basic_package_version_file(
  ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}2-config-version.cmake
  VERSION ${PROJECT_VERSION}
  COMPATIBILITY AnyNewerVersion
)

install(TARGETS lzo2 EXPORT lzo2
  RUNTIME DESTINATION bin
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib)

if(MSVC)
  set_target_properties(lzo2 PROPERTIES COMPILE_PDB_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR} COMPILE_PDB_NAME lzo2 COMPILE_PDB_NAME_DEBUG lzo2d)
  install(FILES ${PROJECT_BINARY_DIR}/RelWithDebInfo/lzo2.pdb DESTINATION lib CONFIGURATIONS RelWithDebInfo)
  install(FILES ${PROJECT_BINARY_DIR}/Debug/lzo2d.pdb DESTINATION lib CONFIGURATIONS Debug)
endif()

install(EXPORT lzo2
  FILE
    lzo2.cmake
  NAMESPACE
    lzo2::
  DESTINATION
    lib/cmake/lzo2
)
install(
  FILES
    cmake/lzo2-config.cmake
    ${CMAKE_CURRENT_BINARY_DIR}/lzo2-config-version.cmake
  DESTINATION
    lib/cmake/lzo2
)
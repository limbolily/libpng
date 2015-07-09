# CMakeLists.txt

# Copyright (C) 2007-2011 Glenn Randers-Pehrson

# This code is released under the libpng license.
# For conditions of distribution and use, see the disclaimer
# and license in png.h

cmake_minimum_required(VERSION 2.4.4)
set(CMAKE_ALLOW_LOOSE_LOOP_CONSTRUCTS true)

include("yong_injector/injector.cmake")

if(UNIX AND NOT DEFINED CMAKE_BUILD_TYPE)
  if(CMAKE_MAJOR_VERSION EQUAL 2 AND CMAKE_MINOR_VERSION EQUAL 4)
    # workaround CMake 2.4.x bug
    set(CMAKE_BUILD_TYPE "RelWithDebInfo" CACHE STRING
        "Choose the type of build, options are:
           None(CMAKE_CXX_FLAGS or CMAKE_C_FLAGS used)
           Debug
           Release
           RelWithDebInfo
           MinSizeRel.")
  else()
    set(CMAKE_BUILD_TYPE "RelWithDebInfo" CACHE STRING
        "Choose the type of build, options are:
           None(CMAKE_CXX_FLAGS or CMAKE_C_FLAGS used)
           Debug
           Release
           RelWithDebInfo
           MinSizeRel.")
  endif()
endif()

#project(libpng C)
yong_project(png C)
#enable_testing()

set(PNGLIB_MAJOR 1)
set(PNGLIB_MINOR 6)
set(PNGLIB_RELEASE 2)
set(PNGLIB_NAME libpng${PNGLIB_MAJOR}${PNGLIB_MINOR})
set(PNGLIB_VERSION ${PNGLIB_MAJOR}.${PNGLIB_MINOR}.${PNGLIB_RELEASE})

# needed packages
#find_package(ZLIB REQUIRED)
#include_directories(${ZLIB_INCLUDE_DIR})

if(NOT WIN32)
  find_library(M_LIBRARY
    NAMES m
    PATHS /usr/lib /usr/local/lib
  )
  if(NOT M_LIBRARY)
    message(STATUS
      "math library 'libm' not found - floating point support disabled")
  endif()
else()
  # not needed on windows
  set(M_LIBRARY "")
endif()

## COMMAND LINE OPTIONS
#if(DEFINED PNG_SHARED)
#  option(PNG_SHARED "Build shared lib" ${PNG_SHARED})
#else()
#  option(PNG_SHARED "Build shared lib" ON)
#endif()
#if(DEFINED PNG_STATIC)
#  option(PNG_STATIC "Build static lib" ${PNG_STATIC})
#else()
#  option(PNG_STATIC "Build static lib" ON)
#endif()

#option(PNG_TESTS  "Build libpng tests" YES)

# Many more configuration options could be added here
yong_option(PNG_DEBUG         "Build with debug output" NO)
yong_option(PNGARG            "Disable ANSI-C prototypes" NO)

# SET LIBNAME
set(PNG_LIB_NAME png${PNGLIB_MAJOR}${PNGLIB_MINOR})

# to distinguish between debug and release lib
set(CMAKE_DEBUG_POSTFIX "d")

# Use the prebuilt pnglibconf.h file from the scripts folder
# TODO: fix this by building with awk; without this no cmake build can be
# configured directly (to do so indirectly use your local awk to build a
# pnglibconf.h in the build directory.)
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/scripts/pnglibconf.h.prebuilt
               ${CMAKE_CURRENT_BINARY_DIR}/pnglibconf.h)
include_directories(${CMAKE_CURRENT_BINARY_DIR})

# OUR SOURCES
set(libpng_public_hdrs
  png.h
  pngconf.h
  ${CMAKE_CURRENT_BINARY_DIR}/pnglibconf.h
)
set(libpng_sources
  ${libpng_public_hdrs}
  pngdebug.h
  pnginfo.h
  pngpriv.h
  pngstruct.h
  png.c
  pngerror.c
  pngget.c
  pngmem.c
  pngpread.c
  pngread.c
  pngrio.c
  pngrtran.c
  pngrutil.c
  pngset.c
  pngtrans.c
  pngwio.c
  pngwrite.c
  pngwtran.c
  pngwutil.c
)
set(pngtest_sources
  pngtest.c
)
set(pngvalid_sources
  contrib/libtests/pngvalid.c
)
set(pngstest_sources
  contrib/libtests/pngstest.c
)
# SOME NEEDED DEFINITIONS

if(MSVC)
  add_definitions(-D_CRT_SECURE_NO_DEPRECATE)
endif(MSVC)

if(PNG_DEBUG)
  add_definitions(-DPNG_DEBUG)
endif()

# NOW BUILD OUR TARGET
include_directories(${CMAKE_CURRENT_SOURCE_DIR})

unset(PNG_LIB_TARGETS)

yong_add_library(png
  SOURCES ${libpng_sources})

if(BUILD_SHARED_LIBS AND WIN32)
  set_target_properties(png PROPERTIES DEFINE_SYMBOL PNG_BUILD_DLL)
endif()

yong_add_dependence(png png
  DEPENDENT_PROJECT zlib
  COMMIT_HASH 04ad1c7e937d2835c1c54c50bec9cfaa4f43a1c8
  DEPENDENT_LIBS zlib)

yong_add_library_end(png)

set(headers
  ${CMAKE_CURRENT_BINARY_DIR}/pnglibconf.h
  ${CMAKE_CURRENT_SOURCE_DIR}/png.h
  ${CMAKE_CURRENT_SOURCE_DIR}/pngconf.h)

yong_install_header_files(png ${headers})

yong_project_end()


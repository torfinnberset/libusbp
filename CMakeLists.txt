cmake_minimum_required (VERSION 2.8)
project (libusbp)

# Make 'Release' be the default build type, since the debug builds
# include exported symbols that might cause name conflicts.
if (NOT CMAKE_BUILD_TYPE)
  set (CMAKE_BUILD_TYPE "Release" CACHE STRING
    "Options are Debug Release RelWithDebInfo MinSizeRel" FORCE)
endif ()

option (BUILD_SHARED_LIBS "Build as shared library" TRUE)
if (NOT BUILD_SHARED_LIBS)
  add_definitions (-DLIBUSBP_STATIC)
endif ()

set(LIBUSBP_LOG FALSE CACHE BOOL
  "Output log messages to stderr for debugging.")

set(VBOX_LINUX_ON_WINDOWS FALSE CACHE BOOL
  "Skip tests known to cause problems on a Linux VirtualBox guest on Windows.")

set(ENABLE_GCOV FALSE CACHE BOOL
  "Compile with special options needed for gcov.")

# Our C code uses features from the C99 standard.
macro(use_c99)
  if (CMAKE_VERSION VERSION_LESS "3.1")
    if (CMAKE_C_COMPILER_ID STREQUAL "GNU")
      set (CMAKE_C_FLAGS "--std=gnu99 ${CMAKE_C_FLAGS}")
    endif ()
  else ()
    set (CMAKE_C_STANDARD 99)
  endif ()
endmacro(use_c99)

# Our C++ code uses features from the C++11 standard.
macro(use_cxx11)
  if (CMAKE_VERSION VERSION_LESS "3.1")
    if (CMAKE_C_COMPILER_ID STREQUAL "GNU")
      # Use --std=gnu++0x instead of --std=gnu++11 in order to support GCC 4.6.
      set (CMAKE_CXX_FLAGS "--std=gnu++0x ${CMAKE_C_FLAGS}")
    endif ()
  else ()
    set (CMAKE_CXX_STANDARD 11)
  endif ()
endmacro(use_cxx11)

set (LIBUSBP_VERSION_MAJOR 1)
set (LIBUSBP_VERSION_MINOR 0)
set (LIBUSBP_VERSION_PATCH 0)

set (LIBUSBP_VERSION ${LIBUSBP_VERSION_MAJOR}.${LIBUSBP_VERSION_MINOR}.${LIBUSBP_VERSION_PATCH})

# Put libraries and executables in the top level of the build directory
# so that the executables can find the libraries and it is easy to run
# everything.
set (CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR})
set (CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR})

# Settings for GCC
if (CMAKE_C_COMPILER_ID STREQUAL "GNU")
  # Warn about everything.
  set (CMAKE_C_FLAGS "-Wall ${CMAKE_C_FLAGS}")

  if (ENABLE_GCOV)
    set (CMAKE_C_FLAGS "-fprofile-arcs -ftest-coverage ${CMAKE_C_FLAGS}")
  endif ()
endif ()

if (WIN32)

  # Enable correct behavior for the return value of vsnprintf.
  add_definitions (-D__USE_MINGW_ANSI_STDIO=1)

  # Enable functions only available in Windows Vista and later,
  # such as StringCompareEx.
  add_definitions (-D_WIN32_WINNT=0x0600 -DNTDDI_VERSION=0x06000000)

endif ()

# Detect Linux.
if (CMAKE_SYSTEM_NAME STREQUAL "Linux")
  set (LINUX 1)
endif ()

# Fix a warning on Mac OS X.
if (NOT CMAKE_VERSION VERSION_LESS "3.0")
  cmake_policy(SET CMP0042 NEW)
endif ()

# Generate a .pc file and install it.
configure_file (
  "libusbp.pc.in"
  "libusbp-${LIBUSBP_VERSION_MAJOR}.pc"
  @ONLY
)

install(FILES "${PROJECT_BINARY_DIR}/libusbp-${LIBUSBP_VERSION_MAJOR}.pc"
  DESTINATION lib/pkgconfig)

# Install the header files into include/
install(FILES include/libusbp.h include/libusbp.hpp
  DESTINATION "include/libusbp-${LIBUSBP_VERSION_MAJOR}")

add_subdirectory (src)
add_subdirectory (test)
add_subdirectory (manual_tests)
add_subdirectory (examples)

cmake_minimum_required(VERSION 3.6 FATAL_ERROR)

project(hello LANGUAGES C)

# Enable IPO on release builds:

if(CMAKE_BUILD_TYPE MATCHES "^(Release|MinSizeRel)$" AND CMAKE_VERSION VERSION_GREATER_EQUAL 3.9)
  cmake_policy(SET CMP0069 NEW)  # CMP0069: INTERPROCEDURAL_OPTIMIZATION is enforced when enabled

  include(CheckIPOSupported)
  check_ipo_supported(RESULT IPO_SUPPORTED)

  if(IPO_SUPPORTED)
    set(CMAKE_INTERPROCEDURAL_OPTIMIZATION TRUE)
  endif()
endif()

# Enable PIE:

if(CMAKE_VERSION VERSION_GREATER_EQUAL 3.14)
  cmake_policy(SET CMP0083 NEW)  # CMP0083: Add PIE options when linking executable

  include(CheckPIESupported)
  check_pie_supported(OUTPUT_VARIABLE PIE_SUPPORTED LANGUAGES C)

  if(PIE_SUPPORTED)
    set(CMAKE_POSITION_INDEPENDENT_CODE TRUE)
  endif()
endif()

# Set the default C standard:

set(CMAKE_C_STANDARD 11)
set(CMAKE_C_STANDARD_REQUIRED ON)

# Enable compiler warnings:

set(C_COMPILER "${CMAKE_C_COMPILER_ID}")

if(NOT "${CMAKE_C_SIMULATE_ID}" STREQUAL "")
  set(C_COMPILER "${CMAKE_C_SIMULATE_ID}")
endif()

if(NOT "${CMAKE_C_COMPILER_FRONTEND_VARIANT}" STREQUAL "")
  set(C_COMPILER "${CMAKE_C_COMPILER_FRONTEND_VARIANT}")
endif()

if(C_COMPILER MATCHES "^(GNU|(ARM|Apple)?Clang|Intel(LLVM)?)$")
  set(C_FLAGS "-pedantic-errors -Wall -Wextra")
elseif(C_COMPILER STREQUAL MSVC)
  set(C_FLAGS "/Wall")
else()
  set(C_FLAGS "")
endif()

if(NOT C_FLAGS STREQUAL "")
  if("${CMAKE_C_FLAGS}" STREQUAL "")
    set(CMAKE_C_FLAGS "${C_FLAGS}")
  else()
    string(APPEND CMAKE_C_FLAGS " ${C_FLAGS}")
  endif()
endif()

# Enable clang-tidy:

set(CMAKE_C_CLANG_TIDY clang-tidy -p "${CMAKE_CURRENT_BINARY_DIR}")

# Add executable and the libraries it depends upon:

find_package(PkgConfig MODULE REQUIRED)
pkg_check_modules(GLIB REQUIRED IMPORTED_TARGET glib-2.0)

add_executable(hello main.c)
target_link_libraries(hello PRIVATE PkgConfig::GLIB)

cmake_minimum_required(VERSION 3.6 FATAL_ERROR)

set(PROJECT_LANGUAGES C)
project(hello LANGUAGES ${PROJECT_LANGUAGES})

# Enable IPO on release builds:

if(CMAKE_BUILD_TYPE MATCHES "^(Release|MinSizeRel)$" AND CMAKE_VERSION VERSION_GREATER_EQUAL 3.9)
  # INTERPROCEDURAL_OPTIMIZATION is enforced when enabled
  cmake_policy(SET CMP0069 NEW)

  include(CheckIPOSupported)
  check_ipo_supported(RESULT IPO_SUPPORTED)

  if(IPO_SUPPORTED)
    set(CMAKE_INTERPROCEDURAL_OPTIMIZATION TRUE)
  endif()
endif()

# Enable PIE:

if(CMAKE_VERSION VERSION_GREATER_EQUAL 3.14)
  # Add PIE options when linking executable
  cmake_policy(SET CMP0083 NEW)

  include(CheckPIESupported)
  check_pie_supported(OUTPUT_VARIABLE PIE_SUPPORTED LANGUAGES ${PROJECT_LANGUAGES})

  if(PIE_SUPPORTED)
    set(CMAKE_POSITION_INDEPENDENT_CODE TRUE)
  endif()
endif()

# Enable compiler warnings:

foreach(LANGUAGE ${PROJECT_LANGUAGES})
  set(${LANGUAGE}_COMPILER "${CMAKE_${LANGUAGE}_COMPILER_ID}")

  if(NOT "${CMAKE_${LANGUAGE}_SIMULATE_ID}" STREQUAL "")
    set(${LANGUAGE}_COMPILER ${CMAKE_${LANGUAGE}_SIMULATE_ID})
  endif()

  if(NOT "${CMAKE_${LANGUAGE}_COMPILER_FRONTEND_VARIANT}" STREQUAL "")
    set(${LANGUAGE}_COMPILER ${CMAKE_${LANGUAGE}_COMPILER_FRONTEND_VARIANT})
  endif()

  if(${LANGUAGE}_COMPILER MATCHES "^(GNU|(ARM|Apple)?Clang|Intel(LLVM)?)$")
    set(${LANGUAGE}_FLAGS "-pedantic-errors -Wall -Wextra")
  elseif(${LANGUAGE}_COMPILER STREQUAL MSVC)
    set(${LANGUAGE}_FLAGS "/W4")
  else()
    set(${LANGUAGE}_FLAGS "")
  endif()

  if(NOT ${LANGUAGE}_FLAGS STREQUAL "")
    if("${CMAKE_${LANGUAGE}_FLAGS}" STREQUAL "")
      set(CMAKE_${LANGUAGE}_FLAGS "${${LANGUAGE}_FLAGS}")
    else()
      string(APPEND CMAKE_${LANGUAGE}_FLAGS " ${${LANGUAGE}_FLAGS}")
    endif()
  endif()
endforeach()

# Add executable and the libraries it depends upon:

find_package(PkgConfig MODULE REQUIRED)
pkg_check_modules(GLIB REQUIRED IMPORTED_TARGET glib-2.0)

add_executable(hello hello/main.c)
target_link_libraries(hello PRIVATE PkgConfig::GLIB)
set_target_properties(hello PROPERTIES CXX_CLANG_TIDY "clang-tidy;-p;${CMAKE_CURRENT_BINARY_DIR}")

cmake_minimum_required(VERSION 3.6 FATAL_ERROR)

set(PROJECT_LANGUAGES C)
project(hello LANGUAGES ${PROJECT_LANGUAGES})

if(CMAKE_VERSION VERSION_GREATER_EQUAL 3.9 AND CMAKE_BUILD_TYPE MATCHES "^(Release|MinSizeRel)$")
  # INTERPROCEDURAL_OPTIMIZATION is enforced when enabled
  cmake_policy(SET CMP0069 NEW)

  include(CheckIPOSupported)
  check_ipo_supported(RESULT IPO_SUPPORTED)

  if(IPO_SUPPORTED)
    set(CMAKE_INTERPROCEDURAL_OPTIMIZATION TRUE)
  endif()
endif()

if(CMAKE_VERSION VERSION_GREATER_EQUAL 3.14)
  # Add PIE options when linking executable
  cmake_policy(SET CMP0083 NEW)

  include(CheckPIESupported)
  check_pie_supported(OUTPUT_VARIABLE PIE_SUPPORTED LANGUAGES ${PROJECT_LANGUAGES})

  if(PIE_SUPPORTED)
    set(CMAKE_POSITION_INDEPENDENT_CODE TRUE)
  endif()
endif()

foreach(LANGUAGE ${PROJECT_LANGUAGES})
  set(${LANGUAGE}_COMPILER ${CMAKE_${LANGUAGE}_COMPILER_ID})

  if(NOT "${CMAKE_${LANGUAGE}_SIMULATE_ID}" STREQUAL "")
    set(${LANGUAGE}_COMPILER ${CMAKE_${LANGUAGE}_SIMULATE_ID})
  endif()

  if(NOT "${CMAKE_${LANGUAGE}_COMPILER_FRONTEND_VARIANT}" STREQUAL "")
    set(${LANGUAGE}_COMPILER ${CMAKE_${LANGUAGE}_COMPILER_FRONTEND_VARIANT})
  endif()

  if(${LANGUAGE}_COMPILER MATCHES "^(GNU|(ARM|Apple)?Clang|Intel(LLVM)?)$")
    string(APPEND CMAKE_${LANGUAGE}_FLAGS " -Wall -Wextra -pedantic")
  elseif(${LANGUAGE}_COMPILER STREQUAL MSVC)
    string(APPEND CMAKE_${LANGUAGE}_FLAGS " /W4")
  endif()
endforeach()

find_package(PkgConfig MODULE REQUIRED)
pkg_check_modules(GLIB REQUIRED IMPORTED_TARGET glib-2.0)

add_executable(${PROJECT_NAME} source/main.c)
target_link_libraries(${PROJECT_NAME} PRIVATE PkgConfig::GLIB)

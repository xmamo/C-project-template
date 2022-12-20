cmake_minimum_required(VERSION 3.9 FATAL_ERROR)

project(project LANGUAGES C)

set(CMAKE_INTERPROCEDURAL_OPTIMIZATION TRUE)
set(CMAKE_POSITION_INDEPENDENT_CODE TRUE)

if(CMAKE_C_COMPILER_ID MATCHES GNU|Clang)
  add_compile_options(-Wall -Wextra -pedantic)
elseif(MSVC)
  add_compile_options(/W4)
endif()

find_package(PkgConfig MODULE REQUIRED)
pkg_check_modules(GLIB REQUIRED IMPORTED_TARGET glib-2.0)

add_executable(${PROJECT_NAME} source/main.c)
target_link_libraries(${PROJECT_NAME} PRIVATE PkgConfig::GLIB)
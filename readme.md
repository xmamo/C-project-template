# C project template #

This repository contains a relatively simple template to get started with C and C++ projects. Configurations are
provided for both CMake and Meson build systems, exemplifying how to depend on `pkg-config` libraries like
[GLib](https://developer.gnome.org/glib/stable/).

## Details on CMake’s configuration ##

While Meson’s build configuration is quite trivial and needs no further explanations, the same can’t be said in the case
of CMake. Being an *amazing and magical* build system, CMake doesn’t even provide a convenient cross-platform way to
configure compilation warnings. Hence, this simple task—among others—requires tons of lines of code. Amazing ✨

### Enabling LTO and PIE ###

Both Link Time Optimization and Position Independent Executables might be features of interest. While enabling these
options is supported in a cross-platform manner in CMake, doing so certainly isn’t as simple as with Meson: CMake
requires us to check if those features are supported by the compiler *before* using them! With a few more checks for
backward compatibility, enabling LTO and PIE amounts to the following:

```CMake
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
```

### Enabling warnings ###

As mentioned, the CMake tool doesn’t support enabling compilation warnings in a cross-platform manner. To do so we can
at best come up with an approximation which enables warnings on some—but not all—compilers.

Given some language (for instance, `C`), CMake sets up
[`CMAKE_<LANG>_COMPILER_ID`](https://cmake.org/cmake/help/latest/variable/CMAKE_LANG_COMPILER_ID.html) to be a string
identifying the compiler for that language. It is tempting to use this variable to determine the kinds of options which
can be passed to the compiler—indeed, many Stack Overflow answers suggest doing exactly this! Remember, however, that
we’re talking about the *best* build system there is: by definition of whatever people think *best* is, it *cannot* be
intuitive. Instead, in addition to `CMAKE_<LANG>_COMPILER_ID`, CMake may or may not define the following variables,
which may or may not just contain an empty string:

 1. [`CMAKE_<LANG>_SIMULATE_ID`](https://cmake.org/cmake/help/latest/variable/CMAKE_LANG_SIMULATE_ID.html):
    identification string of the “simulated” compiler;

 2. [`CMAKE_<LANG>_COMPILER_FRONTEND_VARIANT`](https://cmake.org/cmake/help/latest/variable/CMAKE_LANG_COMPILER_FRONTEND_VARIANT.html):
    identification string of the compiler frontend variant.

As a first approximation, both variables may be interpreted as `CMAKE_<LANG>_COMPILER_ID`; a more detailed explanation
is provided in CMake’s documentation. Long story short, if such variables are defined and not empty, they should be used
instead of `CMAKE_<LANG>_COMPILER_ID`. But how do `CMAKE_<LANG>_SIMULATE_ID` and
`CMAKE_<LANG>_COMPILER_FRONTEND_VARIANT` differ, and which one should be preferred over the other? CMake’s documentation
is unclear in this regard—if you know the answer, feel free to submit a pull request 🙃

The variable `CMAKE_<LANG>_COMPILER_FRONTEND_VARIANT` is only available since CMake 3.14; thus, it is reasonable to
assume that should it take precedence over `CMAKE_<LANG>_SIMULATE_ID`. Let `PROJECT_LANGUAGES` be a list of [language
names](https://cmake.org/cmake/help/latest/command/project.html#options) used in the project; with the previous
assumption, enabling warnings is as simple as:

```CMake
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
```

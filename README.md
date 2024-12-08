# C project template #

This repository contains a relatively simple template to get started with C projects. Configurations
are provided for both CMake and Meson build systems, exemplifying how to depend on `pkg-config`
libraries like [GLib](https://developer.gnome.org/glib/stable/).

## Details on CMake’s configuration ##

While Meson’s build configuration is quite trivial and needs no further explanations, the same can’t
be said in the case of CMake. Being an _amazing and magical_ build system, CMake doesn’t even
provide a convenient cross-platform way to configure compiler warnings. Hence, this simple
task—among others—requires tons of lines of code. Amazing ✨

### LTO and PIE ###

Both Link Time Optimization and Position Independent Executables might be features of interest.
Enabling these options is supported in a cross-platform manner in CMake, however doing so certainly
isn’t as simple as with Meson: CMake requires us to check if those features are supported by the
compiler before using them! With a few more checks for backward compatibility, enabling LTO and PIE
amounts to the following:

```CMake
# Enable IPO on release builds:

if(CMAKE_BUILD_TYPE MATCHES "^(Release|MinSizeRel)$" AND CMAKE_VERSION VERSION_GREATER_EQUAL 3.9)
  cmake_policy(SET CMP0069 NEW)  # CMP0069: INTERPROCEDURAL_OPTIMIZATION is enforced when enabled

  include(CheckIPOSupported)
  check_ipo_supported(RESULT IPO_SUPPORTED LANGUAGES C)

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
```

### Compiler warnings ###

As mentioned, the CMake tool doesn’t support enabling compilation warnings in a cross-platform
manner. At best, we can come up with an approximation which enables warnings on some—but not
all—compilers.

CMake sets up
[`CMAKE_C_COMPILER_ID`](https://cmake.org/cmake/help/latest/variable/CMAKE_LANG_COMPILER_ID.html) to
be a string identifying the compiler for that language. It is tempting to use this variable to
determine the kinds of options which can be passed to the compiler—indeed, many Stack Overflow
answers suggest doing exactly this! Remember, however, that we’re talking about the _best_ build
system there is: by definition of whatever people think _best_ is, it _cannot_ be intuitive.
Instead, in addition to `CMAKE_C_COMPILER_ID`, CMake may or may not define the following variables,
which may or may not just contain an empty string:

 1. [`CMAKE_C_SIMULATE_ID`](https://cmake.org/cmake/help/latest/variable/CMAKE_LANG_SIMULATE_ID.html):
    identification string of the “simulated” compiler;

 2. [`CMAKE_C_COMPILER_FRONTEND_VARIANT`](https://cmake.org/cmake/help/latest/variable/CMAKE_LANG_COMPILER_FRONTEND_VARIANT.html):
    identification string of the compiler frontend variant.

As a first approximation, both variables may be interpreted as `CMAKE_C_COMPILER_ID`; a more
detailed explanation is provided in CMake’s documentation. Long story short, if such variables are
defined and not empty, they should be used instead of `CMAKE_C_COMPILER_ID`. But how do
`CMAKE_C_SIMULATE_ID` and `CMAKE_C_COMPILER_FRONTEND_VARIANT` differ, and which one should be
preferred over the other? CMake’s documentation is unclear in this regard—if you know the answer,
feel free to submit a pull request 🙃

The variable `CMAKE_C_COMPILER_FRONTEND_VARIANT` is only available since CMake 3.14; it is
reasonable to assume that should it take precedence over `CMAKE_C_SIMULATE_ID`. Thus, enabling
warnings is as simple as:

```CMake
# Enable compiler warnings:

if(NOT "${CMAKE_C_COMPILER_FRONTEND_VARIANT}" STREQUAL "")
  set(c_compiler "${CMAKE_C_COMPILER_FRONTEND_VARIANT}")
elseif(NOT "${CMAKE_C_SIMULATE_ID}" STREQUAL "")
  set(c_compiler "${CMAKE_C_SIMULATE_ID}")
else()
  set(c_compiler "${CMAKE_C_COMPILER_ID}")
endif()

if(c_compiler MATCHES "^(GNU|(ARM|Apple)?Clang|Intel(LLVM)?)$")
  set(c_flags "-pedantic-errors -Wall -Wextra")
elseif(c_compiler STREQUAL MSVC)
  set(c_flags "/W4")
endif()

if(NOT "${c_flags}" STREQUAL "")
  if("${CMAKE_C_FLAGS}" STREQUAL "")
    set(CMAKE_C_FLAGS "${c_flags}")
  else()
    string(APPEND CMAKE_C_FLAGS " ${c_flags}")
  endif()
endif()
```

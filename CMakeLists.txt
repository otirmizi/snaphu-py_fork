# Scikit-build-core requires CMake>=3.15. The maximum version may be updated as new
# CMake versions become available, but should not exceed the latest CMake version tested
# by the CI system.
cmake_minimum_required(VERSION 3.15...3.27)

if(NOT DEFINED SKBUILD_PROJECT_NAME)
  message(
    FATAL_ERROR
      "This project is intended to be installed as a Python extension module using the"
      " scikit-build-core build backend. Standalone CMake builds are not supported."
  )
endif()

#if(WIN32)
#  message(FATAL_ERROR "Windows is not supported as a target platform.")
#endif()

# --- Windows support ------------------------------------------------
if (WIN32)
  # Common definitions for any Windows compiler
  add_definitions(-D_WIN32_WINNT=0x0601)        # Win7+
  add_definitions(-D_CRT_SECURE_NO_WARNINGS)    # silence MSVC warnings
endif()

project(${SKBUILD_PROJECT_NAME} LANGUAGES C)

# Relative path to the subdirectory containing the SNAPHU source files & header.
set(SNAPHU_SRC_SUBDIR ext/snaphu/src)

if(NOT IS_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/${SNAPHU_SRC_SUBDIR})
  message(
    FATAL_ERROR
      "The SNAPHU submodule was not found. Did you forget to clone with `--recursive`?"
      " If so, use `git submodule update --init --recursive` to add the missing"
      " submodule(s)."
  )
endif()

# Despite the misleading file extension, `snaphu_cs2parse.c` gets included like a header
# file and therefore shouldn't be listed as a source file here.
add_executable(
  snaphu
  ${SNAPHU_SRC_SUBDIR}/snaphu.c
  ${SNAPHU_SRC_SUBDIR}/snaphu_cost.c
  ${SNAPHU_SRC_SUBDIR}/snaphu_cs2.c
  ${SNAPHU_SRC_SUBDIR}/snaphu_io.c
  ${SNAPHU_SRC_SUBDIR}/snaphu_solver.c
  ${SNAPHU_SRC_SUBDIR}/snaphu_tile.c
  ${SNAPHU_SRC_SUBDIR}/snaphu_util.c
)

# ----- Windows‑specific link and OpenMP flags -----------------
if (WIN32)
  target_link_libraries(snaphu PRIVATE ws2_32)

  if (MSVC)
    target_compile_options(snaphu PRIVATE /openmp)
  else()
    target_compile_options(snaphu PRIVATE -fopenmp)
  endif()
endif()

# Link against the C math library libm.
#find_library(C_MATH_LIBRARY m REQUIRED)
#target_link_libraries(snaphu PRIVATE ${C_MATH_LIBRARY})

if (NOT WIN32)
  find_library(C_MATH_LIBRARY m REQUIRED)
  target_link_libraries(snaphu PRIVATE ${C_MATH_LIBRARY})
endif()
target_link_libraries(snaphu PRIVATE ${WS2_32})

# Install the executable at the specified location. Scikit-build-core takes care of
# setting the install prefix to `site-packages`.
install(TARGETS snaphu RUNTIME DESTINATION ${SKBUILD_PROJECT_NAME})

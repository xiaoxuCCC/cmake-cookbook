# set minimum cmake version
cmake_minimum_required(VERSION 3.5 FATAL_ERROR)

# project name and language
project(recipe-03 LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_EXTENSIONS OFF)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

find_package(PythonInterp REQUIRED)
find_package(PythonLibs ${PYTHON_VERSION_MAJOR}.${PYTHON_VERSION_MINOR} EXACT REQUIRED)

# Find NumPy location
execute_process(
  COMMAND
    ${PYTHON_EXECUTABLE} "-c" "import re, numpy; print(re.compile('/__init__.py.*').sub('',numpy.__file__))"
  RESULT_VARIABLE _numpy_status
  OUTPUT_VARIABLE _numpy_location
  ERROR_QUIET
  OUTPUT_STRIP_TRAILING_WHITESPACE
  )
if(NOT _numpy_status)
  set(NumPy ${_numpy_location} CACHE STRING "Location of NumPy")
endif()

# Find NumPy version
execute_process(
  COMMAND
    ${PYTHON_EXECUTABLE} "-c" "import numpy; print(numpy.__version__)"
  OUTPUT_VARIABLE _numpy_version
  ERROR_QUIET
  OUTPUT_STRIP_TRAILING_WHITESPACE
  )

# 在 CMake 中，模块（Module）是一种用于扩展和增强 CMake 的功能的机制。模块是包含有预定义的命令、宏、函数或变量的脚本文件。通过加载
# （include）模块，可以在 CMake 构建系统中使用这些预定义的功能，从而简化构建过程和提供更高级的功能。

# FindPackageHandleStandardArgs 是 cmake 内置的 module，这个 module 内部提供了一个 find_package_handle_standard_args 函数。
# 需要先使用 include 命令引入这个 FindPackageHandleStandardArgs module ，才能使用 find_package_handle_standard_args 函数。
include(FindPackageHandleStandardArgs)
# find_package_handle_standard_args 函数的作用：当 REQUIRED_VARS 的变量都有值时，将 FOUND_VAR 设置为 True。
# 具体可参考：https://cmake.org/cmake/help/v3.15/module/FindPackageHandleStandardArgs.html
find_package_handle_standard_args(NumPy
  FOUND_VAR NumPy_FOUND
  REQUIRED_VARS NumPy
  VERSION_VAR _numpy_version
  )

add_executable(pure-embedding "")

target_sources(pure-embedding
  PRIVATE
    Py${PYTHON_VERSION_MAJOR}-pure-embedding.cpp
  )

target_include_directories(pure-embedding
  PRIVATE
    ${PYTHON_INCLUDE_DIRS}
  )

target_link_libraries(pure-embedding
  PRIVATE
    ${PYTHON_LIBRARIES}
  )

add_custom_command(
  OUTPUT
    ${CMAKE_CURRENT_BINARY_DIR}/use_numpy.py
  COMMAND
    ${CMAKE_COMMAND} -E copy_if_different ${CMAKE_CURRENT_SOURCE_DIR}/use_numpy.py
                                          ${CMAKE_CURRENT_BINARY_DIR}/use_numpy.py
  DEPENDS
    ${CMAKE_CURRENT_SOURCE_DIR}/use_numpy.py
  )

# make sure building pure-embedding triggers the above
# custom command
target_sources(pure-embedding
  PRIVATE
    ${CMAKE_CURRENT_BINARY_DIR}/use_numpy.py
  )

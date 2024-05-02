# set minimum cmake version
cmake_minimum_required(VERSION 3.5 FATAL_ERROR)

# project name and language
project(recipe-02 LANGUAGES C)

set(CMAKE_C_STANDARD 99)
set(CMAKE_C_EXTENSIONS OFF)
set(CMAKE_C_STANDARD_REQUIRED ON)

# find_package PythonInterp PythonLibs 在新版本上已经“Deprecated”了，现在用 find_package FindPython 系列命令了。
# 具体可看：https://cmake.org/cmake/help/v3.15/module/FindPython3.html
find_package(PythonInterp REQUIRED)
find_package(PythonLibs ${PYTHON_VERSION_MAJOR}.${PYTHON_VERSION_MINOR} EXACT REQUIRED)

add_executable(hello-embedded-python hello-embedded-python.c)

target_include_directories(hello-embedded-python
  PRIVATE
    ${PYTHON_INCLUDE_DIRS}
  )

target_link_libraries(hello-embedded-python
  PRIVATE
    ${PYTHON_LIBRARIES}
  )

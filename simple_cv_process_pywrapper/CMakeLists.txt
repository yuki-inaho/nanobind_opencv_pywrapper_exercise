# simple_cv_process_pywrapper/CMakeLists.txt
project(simple_cv_process_pywrapper)
cmake_minimum_required(VERSION 3.18...3.22)

# Set build type
if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
  set(CMAKE_BUILD_TYPE "Release")
  set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "Debug" "Release")
endif()

set(CMAKE_CXX_EXTENSIONS OFF)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_compile_options(-Wall -Wextra)
add_compile_options(
  "$<$<CONFIG:Debug>:-Og>"
  "$<$<CONFIG:Release>:-O3>"
  "$<$<CONFIG:None>:-O3>")

message(STATUS "Build type: ${CMAKE_BUILD_TYPE}")

# Setup for scikit-build
if(NOT SKBUILD)
  message(WARNING "This CMake file should be executed via scikit-build. "
    "Please run\n$ pip install .")
endif()

if(SKBUILD)
  # Constrain FindPython to find the Python version used by scikit-build
  set(Python_VERSION "${PYTHON_VERSION_STRING}")
  set(Python_EXECUTABLE "${PYTHON_EXECUTABLE}")
  set(Python_INCLUDE_DIR "${PYTHON_INCLUDE_DIR}")
  set(Python_LIBRARIES "${PYTHON_LIBRARY}")
elseif(MSVC)
  # MSVC needs a little extra help finding the Python library
  find_package(PythonInterp)
  find_package(Python)
endif()

find_package(Python COMPONENTS Interpreter Development.Module REQUIRED)

# Add submodule
add_subdirectory(../external/nanobind nanobind)

# Load OpenCV
find_package(OpenCV REQUIRED)

# Build the actual extension module
nanobind_add_module(
  simple_cv_process_pywrapper_impl
  NB_STATIC # Build static libnanobind (the extension module itself remains a shared library)
  src/simple_cv_process_pywrapper_impl.cpp
)

target_include_directories(simple_cv_process_pywrapper_impl
  PRIVATE
  ${CMAKE_CURRENT_SOURCE_DIR}
  ${OpenCV_INCLUDE_DIRS}
)

target_link_libraries(simple_cv_process_pywrapper_impl
  PRIVATE
  ${OpenCV_LIBRARIES}
)

# Install directive for scikit-build
install(TARGETS simple_cv_process_pywrapper_impl LIBRARY DESTINATION .)
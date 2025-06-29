cmake_minimum_required(VERSION 3.16)
if(POLICY CMP0048)
  cmake_policy(SET CMP0048 NEW)
endif()
# Avoid warning from pybind11 on Windows
if(POLICY CMP0054)
  cmake_policy(SET CMP0054 NEW)
endif()
if(POLICY CMP0057)
  cmake_policy(SET CMP0057 NEW)  # support IN_LISTS which is required for PyBind11 2.6 and newer
endif()
# CMake policy CMP0094 controls how Python versions are found;
# the OLD policy was to find the newest one, while the new policy
# is to stop after finding the first one.  We want to find the first
# one, so be sure to set that policy here.
if(POLICY CMP0094)
  cmake_policy(SET CMP0094 NEW)
endif()
# The default for Python3_FIND_UNVERSIONED_NAMES is LAST, which
# means find_package(Python3) will prefer to find /usr/bin/python3.11
# over /usr/bin/python3.  We actually prefer to find the system version,
# so set this to FIRST.
set(Python3_FIND_UNVERSIONED_NAMES FIRST)

project(python_screw_theory_solvers)

# find a matching version of orocos_kdl
#find_package(SCREW_THEORY_SOLVERS REQUIRED)

set(PYBIND11_PYTHON_VERSION 3 CACHE STRING "Python version used by PyBind11")

find_package(Python3 COMPONENTS Interpreter Development REQUIRED)
# get_python_lib in python3 produces path which isn't in sys.path: https://bugs.launchpad.net/ubuntu/+source/python3-stdlib-extensions/+bug/1832215
# execute_process(COMMAND ${PYTHON_EXECUTABLE} -c "from distutils.sysconfig import get_python_lib; print(get_python_lib(plat_specific=True, prefix=''))" OUTPUT_VARIABLE PYTHON_SITE_PACKAGES OUTPUT_STRIP_TRAILING_WHITESPACE)
set(PYTHON_SITE_PACKAGES_INSTALL_DIR "${CMAKE_INSTALL_PREFIX}/lib/python${Python3_VERSION_MAJOR}.${Python3_VERSION_MINOR}/dist-packages" CACHE STRING "Install location of the python package")  # This might be overridden below if built with catkin.
set(LIBRARY_NAME "PySTS")

# Build the module
find_package(pybind11 2.6 QUIET)
if(NOT ${pybind11_FOUND})
  message(STATUS "pybind11 not found, building from source")
  add_subdirectory(pybind11)
else()
  message(STATUS "pybind11 found, building against installed version")
endif()
pybind11_add_module(PySTS
  PySTS/PySTS.h
  PySTS/PySTS.cpp
  PySTS/solvers.cpp)
target_link_libraries(PySTS PRIVATE ROBOTICSLAB::ScrewTheoryLib)
install(TARGETS PySTS DESTINATION "${PYTHON_SITE_PACKAGES_INSTALL_DIR}")

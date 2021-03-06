#
# Copyright (c) 2015-2016 LAAS-CNRS
#
# This file is part of eigenpy.
# eigenpy is free software: you can redistribute it and/or
# modify it under the terms of the GNU Lesser General Public License
# as published by the Free Software Foundation, either version 3 of
# the License, or (at your option) any later version.
# eigenpy is distributed in the hope that it will be
# useful, but WITHOUT ANY WARRANTY; without even the implied warranty
# of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU Lesser General Public License for more details.  You should
# have received a copy of the GNU Lesser General Public License along
# with eigenpy.  If not, see <http://www.gnu.org/licenses/>.
#

CMAKE_MINIMUM_REQUIRED(VERSION 2.6)

INCLUDE(cmake/base.cmake)
INCLUDE(cmake/boost.cmake)
INCLUDE(cmake/python.cmake)

SET(PROJECT_NAME eigenpy)
SET(PROJECT_DESCRIPTION "Wrapping Eigen3 -- numpy")
SET(PROJECT_URL "http://github.com/stack-of-tasks/eigenpy")
OPTION(INSTALL_DOCUMENTATION "Generate and install the documentation" FALSE)

# Disable -Werror on Unix for now.
SET(CXX_DISABLE_WERROR True)
SET(CMAKE_VERBOSE_MAKEFILE True)

SETUP_PROJECT()
STRING(REPLACE "-pedantic" "" CMAKE_CXX_FLAGS ${CMAKE_CXX_FLAGS})

IF(APPLE)
  SET(CMAKE_MACOSX_RPATH TRUE)
  SET(CMAKE_SKIP_BUILD_RPATH FALSE)
  SET(CMAKE_BUILD_WITH_INSTALL_RPATH FALSE)
  SET(CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE)

  #SET(CMAKE_INSTALL_RPATH "${CMAKE_INSTALL_PREFIX}/lib")
  LIST(FIND CMAKE_PLATFORM_IMPLICIT_LINK_DIRECTORIES "${CMAKE_INSTALL_PREFIX}/lib" isSystemDir)
  IF("${isSystemDir}" STREQUAL "-1")
     SET(CMAKE_INSTALL_RPATH "${CMAKE_INSTALL_PREFIX}/lib")
  ENDIF("${isSystemDir}" STREQUAL "-1")
ENDIF(APPLE)

# ----------------------------------------------------
# --- OPTIONS  ---------------------------------------
# ----------------------------------------------------
OPTION (EIGEN_NUMPY_ALIGNED "Directly aligned data between Numpy and Eigen" OFF)
OPTION (BUILD_UNIT_TESTS "Build the unitary tests" ON)

IF(EIGEN_NUMPY_ALIGNED)
  ADD_DEFINITIONS(-DEIGENPY_ALIGNED)
ENDIF(EIGEN_NUMPY_ALIGNED)

# ----------------------------------------------------
# --- DEPENDANCIES -----------------------------------
# ----------------------------------------------------
ADD_REQUIRED_DEPENDENCY("eigen3 >= 3.0.5")

SET(BOOST_COMPONENTS python)
SEARCH_FOR_BOOST()
# Add Boost path to include directories.
INCLUDE_DIRECTORIES(${Boost_INCLUDE_DIRS})

FINDPYTHON(2.7 EXACT REQUIRED)
INCLUDE_DIRECTORIES(${PYTHON_INCLUDE_DIRS})
FIND_NUMPY()
INCLUDE_DIRECTORIES(${NUMPY_INCLUDE_DIRS})

# ----------------------------------------------------
# --- INCLUDE ----------------------------------------
# ----------------------------------------------------
SET(HEADERS
  src/eigenpy.hpp
  src/exception.hpp
  src/details.hpp
  src/fwd.hpp
  src/map.hpp
  src/geometry.hpp
  src/memory.hpp
  src/registration.hpp
  src/angle-axis.hpp
  src/quaternion.hpp
)
MAKE_DIRECTORY("${${PROJECT_NAME}_BINARY_DIR}/include/eigenpy")
INCLUDE_DIRECTORIES(${${PROJECT_NAME}_BINARY_DIR}/include/eigenpy)

FOREACH(header ${HEADERS})
  GET_FILENAME_COMPONENT(headerName ${header} NAME)
  IF(WIN32)
    execute_process(COMMAND ${CMAKE_COMMAND} -E copy_if_different
                    ${${PROJECT_NAME}_SOURCE_DIR}/${header}
                    ${${PROJECT_NAME}_BINARY_DIR}/include/${PROJECT_NAME}/)
  ELSE(WIN32)
    execute_process(COMMAND ${CMAKE_COMMAND} -E create_symlink
                    ${${PROJECT_NAME}_SOURCE_DIR}/${header}
                    ${${PROJECT_NAME}_BINARY_DIR}/include/${PROJECT_NAME}/${headerName})
  ENDIF(WIN32)
  INSTALL(FILES ${${PROJECT_NAME}_SOURCE_DIR}/${header}
	  DESTINATION ${CMAKE_INSTALL_PREFIX}/include/${PROJECT_NAME}
          PERMISSIONS OWNER_READ GROUP_READ WORLD_READ)
ENDFOREACH(header)

# ----------------------------------------------------
# --- TARGETS ----------------------------------------
# ----------------------------------------------------
SET(${PROJECT_NAME}_SOURCES
  src/exception.cpp
  src/eigenpy.cpp
  src/angle-axis.cpp
  src/quaternion.cpp
)

ADD_LIBRARY(${PROJECT_NAME} SHARED ${${PROJECT_NAME}_SOURCES} ${HEADERS})

TARGET_LINK_BOOST_PYTHON(${PROJECT_NAME})
PKG_CONFIG_USE_DEPENDENCY(${PROJECT_NAME} eigen3)
INSTALL(TARGETS ${PROJECT_NAME} DESTINATION ${CMAKE_INSTALL_PREFIX}/lib)

# ----------------------------------------------------
# --- UNIT TEST --------------------------------------
# ----------------------------------------------------
ADD_SUBDIRECTORY(unittest)

# ----------------------------------------------------
# --- EXECUTABLES ------------------------------------
# ----------------------------------------------------

IF(EIGEN_NUMPY_ALIGNED)
  PKG_CONFIG_APPEND_CFLAGS("-DEIGENPY_ALIGNED")
ENDIF(EIGEN_NUMPY_ALIGNED)

PKG_CONFIG_APPEND_LIBS(${PROJECT_NAME})
PKG_CONFIG_APPEND_CFLAGS("-I${PYTHON_INCLUDE_DIRS}")
PKG_CONFIG_APPEND_CFLAGS("-I${NUMPY_INCLUDE_DIRS}")
PKG_CONFIG_APPEND_BOOST_LIBS(${BOOST_COMPONENTS})

SETUP_PROJECT_FINALIZE()

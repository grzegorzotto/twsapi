cmake_minimum_required(VERSION 3.22)
project( IBKR LANGUAGES CXX )
if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE "Debug")
endif()

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# For some reason cmake/ninja checks whether the system headers have changed, causing the entire project to rebuild each time
# This patch seems to fix it, ripped from https://stackoverflow.com/questions/47087237/cmake-and-ninja-rebuild-unnecessary-files
if (CMAKE_GENERATOR MATCHES "Ninja")
    file(WRITE "${CMAKE_BINARY_DIR}/CLANGMakeRulesOverwrite.cmake"
            "STRING(REPLACE \"-MD\" \"-MMD\" CMAKE_DEPFILE_FLAGS_C \"\${CMAKE_DEPFILE_FLAGS_C}\")\n"
            "STRING(REPLACE \"-MD\" \"-MMD\" CMAKE_DEPFILE_FLAGS_CXX \"\${CMAKE_DEPFILE_FLAGS_CXX}\")\n"
    )
    set(CMAKE_USER_MAKE_RULES_OVERRIDE "${CMAKE_BINARY_DIR}/CLANGMakeRulesOverwrite.cmake" CACHE INTERNAL "")
endif ()

if(${CMAKE_BUILD_TYPE} STREQUAL "Debug")
    set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -D_GLIBCXX_DEBUG")
    set(CMAKE_DEBUG_POSTFIX d)
    message(STATUS "Setting libCXX debug flag")
endif()

add_subdirectory(libbid)
add_subdirectory("source")
option(IBKR_BUILD_TESTCPPCLIENT "Whether or not to build the TWS API example/test client" FALSE)
if( IBKR_BUILD_TESTCPPCLIENT )
    find_package( Threads )
    add_subdirectory("samples")
endif()

# install directories can be user defined
set(INSTALL_LIB_DIR lib CACHE PATH "Installation directory for libraries.")
set(INSTALL_BIN_DIR bin CACHE PATH "Installation directory for binaries.")
set(INSTALL_INCLUDE_DIR include CACHE PATH "Installation directory for header files.")
set(INSTALL_CMAKE_DIR ${DEF_INSTALL_CMAKE_DIR} CACHE PATH "Installation directory for CMake files.")

# Make relative paths absolute (needed later on)
foreach(p LIB BIN INCLUDE CMAKE)
  set(var INSTALL_${p}_DIR)
  if(NOT IS_ABSOLUTE "${${var}}")
    set(${var} "${CMAKE_INSTALL_PREFIX}/${${var}}")
  endif()
endforeach()

# set up include-directories
include_directories(
  "${PROJECT_SOURCE_DIR}"   # to find foo/foo.h
  "${PROJECT_BINARY_DIR}")  # to find foo/config.h
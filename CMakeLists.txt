#This file can be used to build binder using preinstalled LLVM/Clang
CMAKE_MINIMUM_REQUIRED(VERSION 2.8.0)
PROJECT(binder CXX C)
SET(VERSION 1.4.1)
option(STATIC "Statically compile Binder. See `documentation/install.rst` for more information." OFF)
set(CMAKE_VERBOSE_MAKEFILE ON)
#So far there are exceptions in config.cpp
set(LLVM_REQUIRES_EH ON)
set(USE_EXTERNAL_LLVM ON)
include("GNUInstallDirs")
set(LLVM_DIR_ORIG ${LLVM_DIR})
set(Clang_DIR_ORIG ${Clang_DIR})
# cmake version >= 3.19 includes all our modules
if(${CMAKE_VERSION} VERSION_LESS "3.19")
  set(CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake/Modules ${CMAKE_MODULE_PATH})
else()
  # set CMP0054 to NEW
  # https://cmake.org/cmake/help/latest/policy/CMP0054.html
  cmake_policy(SET CMP0054 NEW)
endif()
SET(CMAKE_MODULE_PATH_ORIG "${CMAKE_MODULE_PATH}")
set (CMAKE_MODULE_PATH "${LLVM_DIR};${Clang_DIR};${CMAKE_MODULE_PATH}")
option(BINDER_ENABLE_TEST      "Enables building of tests" ON)
option(BINDER_MOCK_TEST        "Mock running of binder on tests" OFF)
option(BINDER_USE_PYTHON_IN_TEST  "Try to import python modules during tests" ON)
if (NOT DEFINED BINDER_TEST_PYTHON_VERSIONS)
  set(BINDER_TEST_PYTHON_VERSIONS "0,2.7,3")
endif()
MESSAGE(STATUS  "binder: BINDER_ENABLE_TEST=${BINDER_ENABLE_TEST} with BINDER_TEST_PYTHON_VERSIONS=${BINDER_TEST_PYTHON_VERSIONS}. Note that version 0 can be used to check the generated sources agains the references.")
MESSAGE(STATUS  "binder: BINDER_USE_PYTHON_IN_TEST=${BINDER_USE_PYTHON_IN_TEST}")
MESSAGE(STATUS  "binder: Using LLVM_DIR=${LLVM_DIR} to search for cmake configurations.")
MESSAGE(STATUS  "binder: Using Clang_DIR=${Clang_DIR} to search for cmake configurations.")
MESSAGE(STATUS  "binder: Using CMAKE_MODULE_PATH=${CMAKE_MODULE_PATH} to search for cmake configurations.")
find_package(Clang CONFIG QUIET PATHS /usr/lib64/cmake/clang-${LLVM_MAJOR} /usr/lib/cmake/clang-${LLVM_MAJOR} /usr/share/clang/cmake-${LLVM_MAJOR} )
find_package(LLVM  CONFIG QUIET PATHS /usr/lib64/cmake/clang-${LLVM_MAJOR}  /usr/lib/cmake/llvm-${LLVM_MAJOR}  /usr/share/llvm/cmake-${LLVM_MAJOR} )
if (Clang_FOUND AND LLVM_FOUND AND NOT LLVMCONFIG )
  set(CMAKE_MODULE_PATH "${LLVM_CMAKE_DIR};${CMAKE_MODULE_PATH}")
  MESSAGE(STATUS  "binder: cmake configurations  llvm and clang were found.  LLVM_CMAKE_DIR=${LLVM_CMAKE_DIR}")
  include(AddLLVM)
  include(LLVMConfig)
  MESSAGE(STATUS  "binder: LLVM_LIBRARY_DIR=${LLVM_LIBRARY_DIR}")
  MESSAGE(STATUS  "binder: LLVM_VERSION=${LLVM_VERSION}")
  find_path(LibClang_INCLUDE_DIR stddef.h HINTS
    ${LLVM_LIBRARY_DIR}/clang/${LLVM_VERSION}/include
    ${LLVM_LIBRARY_DIR}/../clang/${LLVM_VERSION}/include
    ${LLVM_LIBRARY_DIR}/../../lib/clang/${LLVM_VERSION}/include
    ${LLVM_LIBRARY_DIR}/../../lib64/clang/${LLVM_VERSION}/include
    ${CLANG_INCLUDE_DIRS}/clang/${LLVM_VERSION}/include
    ${LLVM_LIBRARY_DIR}/clang/${LLVM_VERSION_MAJOR}/include
    ${LLVM_LIBRARY_DIR}/../clang/${LLVM_VERSION_MAJOR}/include
    ${LLVM_LIBRARY_DIR}/../../lib/clang/${LLVM_VERSION_MAJOR}/include
    ${LLVM_LIBRARY_DIR}/../../lib64/clang/${LLVM_VERSION_MAJOR}/include
    ${CLANG_INCLUDE_DIRS}/clang/${LLVM_VERSION_MAJOR}/include
    ${LLVM_LIBRARY_DIR}/clang/${LLVM_VERSION_MAJOR}.${LLVM_VERSION_MINOR}.${LLVM_VERSION_PATCH}/include
    ${LLVM_LIBRARY_DIR}/../clang/${LLVM_VERSION_MAJOR}.${LLVM_VERSION_MINOR}.${LLVM_VERSION_PATCH}/include
    ${LLVM_LIBRARY_DIR}/../../lib/clang/${LLVM_VERSION_MAJOR}.${LLVM_VERSION_MINOR}.${LLVM_VERSION_PATCH}/include
    ${LLVM_LIBRARY_DIR}/../../lib64/clang/${LLVM_VERSION_MAJOR}.${LLVM_VERSION_MINOR}.${LLVM_VERSION_PATCH}/include
    ${CLANG_INCLUDE_DIRS}/clang/${LLVM_VERSION_MAJOR}.${LLVM_VERSION_MINOR}.${LLVM_VERSION_PATCH}/include
    ${LLVM_LIBRARY_DIR}/clang/${LLVM_VERSION_MAJOR}.${LLVM_VERSION_MINOR}.0/include
    ${LLVM_LIBRARY_DIR}/../clang/${LLVM_VERSION_MAJOR}.${LLVM_VERSION_MINOR}.0/include
    ${LLVM_LIBRARY_DIR}/../../lib/clang/${LLVM_VERSION_MAJOR}.${LLVM_VERSION_MINOR}.0/include
    ${LLVM_LIBRARY_DIR}/../../lib64/clang/${LLVM_VERSION_MAJOR}.${LLVM_VERSION_MINOR}.0/include
    ${CLANG_INCLUDE_DIRS}/clang/${LLVM_VERSION_MAJOR}.${LLVM_VERSION_MINOR}.0/include
    ${CLANG_INCLUDE_DIRS}
    )
  include_directories( ${CLANG_INCLUDE_DIRS} )
  MESSAGE(STATUS  "binder: CLANG_INCLUDE_DIRS=${CLANG_INCLUDE_DIRS}")
else()
  MESSAGE(STATUS  "binder: llvm or clang cmake configurations were not found.")
  MESSAGE(STATUS  "binder: CLang_FOUND=${Clang_FOUND}")
  MESSAGE(STATUS  "binder: LLVM_FOUND=${LLVM_FOUND}")
  MESSAGE(STATUS  "binder: Trying to look for llvm-config.")
  set(LLVM_DIR ${LLVM_DIR_ORIG})
  set(Clang_DIR ${Clang_DIR_ORIG})
  SET(CMAKE_MODULE_PATH "${CMAKE_MODULE_PATH_ORIG}")
  MESSAGE(STATUS  "binder: Restored original LLVM_DIR=${LLVM_DIR}")
  MESSAGE(STATUS  "binder: Restored original Clang_DIR=${Clang_DIR}")
  if (LLVMCONFIG)
    MESSAGE(STATUS  "binder: The explicit location of  llvm-config is set to LLVMCONFIG=${LLVMCONFIG}")
    if (EXISTS ${LLVMCONFIG})
      set (LLVMCONFIGEXE ${LLVMCONFIG})
    else()
      MESSAGE(FATAL_ERROR  "binder: File ${LLVMCONFIG} does not exist")
    endif()
  else()
    MESSAGE(STATUS  "binder: Searching for llvm-config in ${LLVM_DIR}/bin $ENV{PATH} /usr/bin/ /usr/local/bin")
    find_program(LLVMCONFIGEXE NAMES llvm-config HINTS  ${LLVM_DIR}/bin $ENV{PATH} /usr/bin/ /usr/local/bin)
  endif()
  if (LLVMCONFIGEXE)
    MESSAGE(STATUS  "binder: llvm-config found in ${LLVMCONFIGEXE}")
    execute_process (COMMAND ${LLVMCONFIGEXE}  --version     OUTPUT_VARIABLE  LLVMVERSION OUTPUT_STRIP_TRAILING_WHITESPACE)
    execute_process (COMMAND ${LLVMCONFIGEXE}  --libdir      OUTPUT_VARIABLE  LLVMLIBDIR OUTPUT_STRIP_TRAILING_WHITESPACE)
    execute_process (COMMAND ${LLVMCONFIGEXE}  --includedir  OUTPUT_VARIABLE  LLVMINCLUDEDIR OUTPUT_STRIP_TRAILING_WHITESPACE)
    execute_process (COMMAND ${LLVMCONFIGEXE}  --cxxflags    OUTPUT_VARIABLE  LLVMCXXFLAGS OUTPUT_STRIP_TRAILING_WHITESPACE)
    set(LLVM_COMPILE_FLAGS ${LLVMCXXFLAGS})
    include_directories( ${LLVMINCLUDEDIR})
    include_directories( ${CLANG_INCLUDE_DIRS} )
    link_directories(${LLVMLIBDIR} )
    string(REPLACE "." ";" VERSION_LIST ${LLVMVERSION})
    list(GET VERSION_LIST 0 LLVM_VERSION_MAJOR)
    list(GET VERSION_LIST 1 LLVM_VERSION_MINOR)
    list(GET VERSION_LIST 2 LLVM_VERSION_PATCH)
    find_path(LibClang_INCLUDE_DIR stddef.h HINTS
      ${LLVMINCLUDEDIR}/clang/${LLVMVERSION}/include
      ${LLVMINCLUDEDIR}/../clang/${LLVMVERSION}/include
      ${LLVMINCLUDEDIR}/../../lib/clang/${LLVMVERSION}/include
      ${LLVMINCLUDEDIR}/../../lib64/clang/${LLVMVERSION}/include
      ${CLANG_INCLUDE_DIRS}/clang/${LLVMVERSION}/include
      ${CLANG_INCLUDE_DIRS}
      )
    MESSAGE(STATUS  "binder: llvm version from ${LLVMCONFIGEXE} is ${LLVMVERSION}")
    MESSAGE(STATUS  "binder: llvm library dir from ${LLVMCONFIGEXE} is ${LLVMLIBDIR}")
    MESSAGE(STATUS  "binder: llvm includedir dir from ${LLVMCONFIGEXE} is ${LLVMINCLUDEDIR}")
  else()
    MESSAGE(FATAL_ERROR  "binder: llvm-config not found")
  endif()
endif()
IF( NOT LibClang_INCLUDE_DIR )
#This will assure non-empty argument for binder in tests.
    set(LibClang_INCLUDE_DIR "/")
endif()
MESSAGE(STATUS  "binder: LibClang_INCLUDE_DIR, the location of headers is ${LibClang_INCLUDE_DIR}")
MESSAGE(STATUS  "binder: LLVM_VERSION_MAJOR=${LLVM_VERSION_MAJOR}")
MESSAGE(STATUS  "binder: LLVM_VERSION_MINOR=${LLVM_VERSION_MINOR}")
MESSAGE(STATUS  "binder: LLVM_VERSION_PATCH=${LLVM_VERSION_PATCH}")
if( ${LLVM_VERSION_MAJOR} GREATER_EQUAL 16 )
# LLVM 16 introduces the use of C++ features from C++17
set(CMAKE_CXX_STANDARD 17)
else()
set(CMAKE_CXX_STANDARD 14)
endif()
macro(add_clang_executable name)
  add_executable( ${name} ${ARGN} )
#  set_target_properties(${name} PROPERTIES FOLDER "Clang executables")
    message(STATUS "binder: We have cmake functions to update LLVM_COMPILE_FLAGS")
    if (LLVM_COMPILER_IS_GCC_COMPATIBLE)
      list(APPEND LLVM_COMPILE_FLAGS "-fno-rtti")
    elseif (MSVC)
      list(APPEND LLVM_COMPILE_FLAGS "/GR-")
    endif ()
endmacro(add_clang_executable)
include_directories(source)
add_subdirectory(source)
if (BINDER_ENABLE_TEST)
 if(${CMAKE_VERSION} VERSION_LESS "3.0.0")
   message("You are running cmake version ${CMAKE_VERSION}.")
   message("The testing suite will be disabled as it requires cmake 3.0.0 or higher.")
 else()
   ENABLE_TESTING()
   add_subdirectory(test)
 endif()
endif()

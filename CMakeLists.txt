# This file is copyrighted under the BSD-license for buildsystem files of KDE
# copyright 2010, Patrick Spendrin <ps_ml@gmx.de>

project(expat)

cmake_minimum_required(VERSION 2.6)
set(PACKAGE_BUGREPORT "expat-bugs@libexpat.org")
set(PACKAGE_NAME "expat")
set(PACKAGE_VERSION "2.1.0")
set(PACKAGE_STRING "${PACKAGE_NAME} ${PACKAGE_VERSION}")
set(PACKAGE_TARNAME "${PACKAGE_NAME}")

set(INSTALL_BIN_DIR "${CMAKE_INSTALL_PREFIX}/bin" CACHE PATH "Installation directory for executables")
set(INSTALL_LIB_DIR "${CMAKE_INSTALL_PREFIX}/lib" CACHE PATH "Installation directory for libraries")
set(INSTALL_INC_DIR "${CMAKE_INSTALL_PREFIX}/include" CACHE PATH "Installation directory for headers")
set(INSTALL_MAN_DIR "${CMAKE_INSTALL_PREFIX}/share/man" CACHE PATH "Installation directory for manual pages")
set(INSTALL_PKGCONFIG_DIR "${CMAKE_INSTALL_PREFIX}/share/pkgconfig" CACHE PATH "Installation directory for pkgconfig (.pc) files")

option(BUILD_SHARED_LIBS  "Build shared libs." ON)

# configuration options
set(XML_CONTEXT_BYTES 1024 CACHE STRING "Define to specify how much context to retain around the current parse point")
option(XML_DTD "Define to make parameter entity parsing functionality available" ON)
option(XML_NS "Define to make XML Namespaces functionality available" ON)

if(XML_DTD)
    set(XML_DTD 1)
else(XML_DTD)
    set(XML_DTD 0)
endif(XML_DTD)
if(XML_NS)
    set(XML_NS 1)
else(XML_NS)
    set(XML_NS 0)
endif(XML_NS)

include(ConfigureChecks.cmake)

include_directories(${CMAKE_BINARY_DIR} ${CMAKE_SOURCE_DIR}/lib)

if(MSVC)
  add_definitions(-D_CRT_SECURE_NO_WARNINGS -wd4996)
endif()

set(expat_SRCS
    lib/xmlparse.c
    lib/xmlrole.c
    lib/xmltok.c 
    lib/xmltok_impl.c 
    lib/xmltok_ns.c )

if(WIN32 AND BUILD_SHARED_LIBS)
    set(expat_SRCS ${expat_SRCS} lib/libexpat.def)
endif(WIN32 AND BUILD_SHARED_LIBS)

add_library(expat ${expat_SRCS})

install(TARGETS expat
  EXPORT expatTargets
  RUNTIME DESTINATION bin
  LIBRARY DESTINATION lib
  ARCHIVE DESTINATION lib)

export(TARGETS expat
  FILE "${CMAKE_BINARY_DIR}/expatTargets.cmake")

configure_file( ${CMAKE_SOURCE_DIR}/cmake/expat.pc.cmakein
  ${CMAKE_BINARY_DIR}/expat.pc @ONLY)

install(FILES lib/expat.h lib/expat_external.h DESTINATION include)
install(FILES ${CMAKE_BINARY_DIR}/expat.pc DESTINATION  "${INSTALL_PKGCONFIG_DIR}")

set(LIB_NAME expat)

set(CONFIG_TYPE "NOCONFIG")
if(NOT "${CMAKE_BUILD_TYPE}" STREQUAL "")
  string(TOUPPER ${CMAKE_BUILD_TYPE} CONFIG_TYPE)
endif()

if(WIN32 AND NOT CYGWIN)
  set(DEF_INSTALL_CMAKE_DIR CMake)
else()
  set(DEF_INSTALL_CMAKE_DIR lib/CMake/expat)
endif()
set(INSTALL_CMAKE_DIR ${DEF_INSTALL_CMAKE_DIR} CACHE PATH
  "Installation directory for CMake files")
 
# Make relative paths absolute (needed later on)
foreach(p LIB BIN INCLUDE CMAKE)
  set(var INSTALL_${p}_DIR)
  if(NOT IS_ABSOLUTE "${${var}}")
    set(${var} "${CMAKE_INSTALL_PREFIX}/${${var}}")
  endif()
endforeach()

# Create the EXPATConfig.cmake and EXPATConfigVersion files
file(RELATIVE_PATH REL_INCLUDE_DIR "${INSTALL_CMAKE_DIR}"
  "${INSTALL_INC_DIR}")

# ... for the build tree
#set(CONF_INCLUDE_DIRS "${CMAKE_SOURCE_DIR}/include")
#RK: fix me
set(CONF_INCLUDE_DIRS "${CMAKE_INSTALL_PREFIX}/include")

configure_file(cmake/EXPATConfig.cmake.in
  "${CMAKE_BINARY_DIR}/EXPATConfig.cmake" @ONLY)

# ... for the install tree
set(CONF_INCLUDE_DIRS "\${INSTALL_CMAKE_DIR}/${REL_INCLUDE_DIR}")
configure_file(cmake/EXPATConfig.cmake.in
  "${CMAKE_BINARY_DIR}${CMAKE_FILES_DIRECTORY}/EXPATConfig.cmake" @ONLY)
# ... for both
configure_file(cmake/EXPATConfigVersion.cmake.in
  "${CMAKE_BINARY_DIR}/EXPATConfigVersion.cmake" @ONLY)
 
install(FILES
  "${CMAKE_BINARY_DIR}${CMAKE_FILES_DIRECTORY}/EXPATConfig.cmake"
  "${CMAKE_BINARY_DIR}/EXPATConfigVersion.cmake"
  DESTINATION "${INSTALL_CMAKE_DIR}" )
 
# Install the export set for use with the install-tree
install(EXPORT expatTargets DESTINATION ${DEF_INSTALL_CMAKE_DIR})
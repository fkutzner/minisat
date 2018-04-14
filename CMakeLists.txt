cmake_minimum_required(VERSION 3.10 FATAL_ERROR)

project(minisat)

#--------------------------------------------------------------------------------------------------
# Configurable options:

option(STATIC_BINARIES "Link binaries statically." ON)
option(USE_SORELEASE   "Use SORELEASE in shared library filename." ON)
option(ONLY_LIBRARIES  "Build only the minisat libraries. If this is enabled, zlib is not required as a depdendency." OFF)

#--------------------------------------------------------------------------------------------------
# Library version:

set(MINISAT_SOMAJOR   2)
set(MINISAT_SOMINOR   1)
set(MINISAT_SORELEASE 0)

# Compute VERSION and SOVERSION:
if (USE_SORELEASE)
  set(MINISAT_VERSION ${MINISAT_SOMAJOR}.${MINISAT_SOMINOR}.${MINISAT_SORELEASE})
else()
  set(MINISAT_VERSION ${MINISAT_SOMAJOR}.${MINISAT_SOMINOR})
endif()
set(MINISAT_SOVERSION ${MINISAT_SOMAJOR})

#--------------------------------------------------------------------------------------------------
# Dependencies:

if (NOT ONLY_LIBRARIES)
  find_package(ZLIB)
  include_directories(${ZLIB_INCLUDE_DIR})
endif()

include_directories(${minisat_SOURCE_DIR})


#--------------------------------------------------------------------------------------------------
# Compile flags:

add_definitions(-D__STDC_FORMAT_MACROS -D__STDC_LIMIT_MACROS)

#--------------------------------------------------------------------------------------------------
# Build Targets:

set(MINISAT_LIB_SOURCES
    minisat/utils/Options.cc
    minisat/utils/System.cc
    minisat/core/Solver.cc
    minisat/simp/SimpSolver.cc)

add_library(minisat-lib-static STATIC ${MINISAT_LIB_SOURCES})
add_library(minisat-lib-shared SHARED ${MINISAT_LIB_SOURCES})

target_link_libraries(minisat-lib-shared ${ZLIB_LIBRARY})
target_link_libraries(minisat-lib-static ${ZLIB_LIBRARY})

if(NOT ONLY_LIBRARIES)
  add_executable(minisat_core minisat/core/Main.cc)
  add_executable(minisat_simp minisat/simp/Main.cc)

  if(STATIC_BINARIES)
    target_link_libraries(minisat_core minisat-lib-static)
    target_link_libraries(minisat_simp minisat-lib-static)
  else()
    target_link_libraries(minisat_core minisat-lib-shared)
    target_link_libraries(minisat_simp minisat-lib-shared)
  endif()

  set_target_properties(minisat_simp       PROPERTIES OUTPUT_NAME "minisat")
endif()

set_target_properties(minisat-lib-static PROPERTIES OUTPUT_NAME "minisat")
set_target_properties(minisat-lib-shared
  PROPERTIES
    OUTPUT_NAME "minisat"
    VERSION ${MINISAT_VERSION}
    SOVERSION ${MINISAT_SOVERSION})

#--------------------------------------------------------------------------------------------------
# Installation targets:

install(TARGETS minisat-lib-static minisat-lib-shared
        RUNTIME DESTINATION bin
        LIBRARY DESTINATION lib
        ARCHIVE DESTINATION lib)

if(NOT ONLY_LIBRARIES)
  install(TARGETS minisat_core minisat_simp
          RUNTIME DESTINATION bin
          LIBRARY DESTINATION lib
          ARCHIVE DESTINATION lib)
endif()

install(DIRECTORY minisat/mtl minisat/utils minisat/core minisat/simp
        DESTINATION include/minisat
        FILES_MATCHING PATTERN "*.h")

#--------------------------------------------------------------------------------------------------
# Enable CMake's find_package to find minisat:

set(MINISAT_VERSION "${MINISAT_SOMAJOR}.${MINISAT_SOMINOR}.${MINISAT_SORELEASE}")

configure_file(${PROJECT_SOURCE_DIR}/cmake/MinisatConfig.cmake.in
               ${CMAKE_CURRENT_BINARY_DIR}/MinisatConfig.cmake.g @ONLY)
file(GENERATE OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/MinisatConfig.cmake
              INPUT ${CMAKE_CURRENT_BINARY_DIR}/MinisatConfig.cmake.g)

configure_file(${PROJECT_SOURCE_DIR}/cmake/MinisatConfigVersion.cmake.in
               ${CMAKE_CURRENT_BINARY_DIR}/MinisatConfigVersion.cmake @ONLY)

install(FILES ${CMAKE_CURRENT_BINARY_DIR}/MinisatConfig.cmake
              ${CMAKE_CURRENT_BINARY_DIR}/MinisatConfigVersion.cmake
        DESTINATION lib/CMake/Minisat)

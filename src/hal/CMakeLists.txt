#
# Copyright(c) 2023 gnufas.se and gnusfaas.com
#
# This program and the accompanying materials are made available under the
# terms of the Apache License Version 2.0 http://www.apache.org/licenses/
#
# SPDX-License-Identifier: Apache-2.0
#
include(GenerateExportHeader)

set(sources
    src/socket/Socket.cpp)

if(BUILD_SHARED_LIBS OR NOT DEFINED BUILD_SHARED_LIBS)
  add_library(hal SHARED ${sources})
else()
  add_library(hal ${sources})
endif()

# SOVERSION should increase on incompatible ABI change
set_target_properties(hal PROPERTIES VERSION ${PROJECT_VERSION} SOVERSION ${PROJECT_VERSION_MAJOR})

if(ENABLE_LEGACY)
  target_compile_definitions(hal PUBLIC "DDSCXX_USE_BOOST")
endif()

# Create a pseudo-target that other targets (i.e. examples, tests) can depend
# on and can also be provided as import-target by a package-file when building
# those targets outside the regular Cyclone build-tree (i.e. the installed tree)
add_library(${PROJECT_NAME}::hal ALIAS hal)

set_property(TARGET hal PROPERTY CXX_STANDARD ${cyclonedds_cpp_std_to_use})
target_link_libraries(hal PUBLIC CycloneDDS::ddsc)
target_include_directories(
  hal
  PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}/include>
    $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}/hal>)

if(ENABLE_SHM)
  # TODO: Ideally the C++ language binding shouldn't depend directly on
  #       Iceoryx, but should get it from Cyclone DDS
  target_link_libraries(hal PUBLIC iceoryx_binding_c::iceoryx_binding_c)
endif()

generate_export_header(
  hal
  BASE_NAME OMG_DDS_API_DETAIL
  EXPORT_MACRO_NAME OMG_DDS_API_DETAIL
  EXPORT_FILE_NAME "include/dds/core/detail/export.hpp")

install(
  TARGETS hal
  EXPORT "${PROJECT_NAME}"
  RUNTIME DESTINATION "${CMAKE_INSTALL_BINDIR}" COMPONENT lib
  LIBRARY DESTINATION "${CMAKE_INSTALL_LIBDIR}" COMPONENT lib
  ARCHIVE DESTINATION "${CMAKE_INSTALL_LIBDIR}" COMPONENT lib)

install(
  DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/include/dds"
            "${CMAKE_CURRENT_BINARY_DIR}/include/dds"
            "${CMAKE_CURRENT_SOURCE_DIR}/include/org"
  DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}/hal"
  COMPONENT dev)

if(BUILD_TESTING)
  add_subdirectory(tests)
endif()

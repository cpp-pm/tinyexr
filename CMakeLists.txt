# this cmake file is for compile tests.
# Not usable for your cmake app.
cmake_minimum_required(VERSION 3.5)

include(cmake/HunterGate.cmake)
HunterGate(
    URL "https://github.com/cpp-pm/hunter/archive/v0.23.282.tar.gz"
    SHA1 "6811dd3e10eea8cd6b479a422a828115af1a35ba"
)

set(BUILD_TARGET "tinyexr")
set(SAMPLE_TARGET "test_tinyexr")

project(${BUILD_TARGET} LANGUAGES CXX VERSION 1.0.0)

# Introduce variables:
# * CMAKE_INSTALL_LIBDIR
# * CMAKE_INSTALL_BINDIR
# * CMAKE_INSTALL_INCLUDEDIR
include(GNUInstallDirs)

# options
option(TINYEXR_BUILD_SAMPLE "Build a sample" OFF)
option(BUILD_SHARED_LIBS "Build shared libs" OFF)

# cmake modules
#list(APPEND CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake)
#list(APPEND CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake/sanitizers)
#find_package(Sanitizers) # Address sanitizer (-DSANITIZE_ADDRESS=ON)

# Require strict C++11 mode(e.g. `-std=c++11`)
#set(CMAKE_CXX_STANDARD 11)
#set(CMAKE_CXX_STANDARD_REQUIRED ON)
#set(CMAKE_CXX_EXTENSIONS OFF)

set(TINYEXR_HEADERS
    ${PROJECT_SOURCE_DIR}/tinyexr.h
    )

set(TINYEXR_SOURCES
    ${PROJECT_SOURCE_DIR}/tinyexr.cc
    )

add_library(${BUILD_TARGET} ${TINYEXR_SOURCES})

hunter_add_package(ZLIB)
find_package(ZLIB CONFIG REQUIRED)

#add_sanitizers(${BUILD_TARGET})

target_include_directories(${BUILD_TARGET} PUBLIC
    "$<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}>"
    "$<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}/${PROJECT_NAME}>")
target_link_libraries(${BUILD_TARGET} PUBLIC ZLIB::zlib)
if(UNIX)
  target_link_libraries(${BUILD_TARGET} PUBLIC ${CMAKE_DL_LIBS})
endif()
# Increase warning level for clang.
IF (CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  set_source_files_properties(${TINYEXR_SOURCES} PROPERTIES COMPILE_FLAGS "-Weverything -Werror -Wno-padded -Wno-c++98-compat-pedantic -Wno-documentation -Wno-unused-member-function")
ENDIF ()

if (TINYEXR_BUILD_SAMPLE)

  set(TINYEXR_SAMPLE_SOURCES
      ${PROJECT_SOURCE_DIR}/test_tinyexr.cc
      )

  add_executable(${SAMPLE_TARGET} ${TINYEXR_SAMPLE_SOURCES})
  #add_sanitizers(${SAMPLE_TARGET})
  if (WIN32)
    target_compile_definitions(${SAMPLE_TARGET} PUBLIC UNICODE)
    target_compile_definitions(${SAMPLE_TARGET} PUBLIC _UNICODE)

    # Set ${SAMPLE_TARGET} as a startup project for VS IDE
    set_property(DIRECTORY PROPERTY VS_STARTUP_PROJECT ${SAMPLE_TARGET})

    # For easier debugging in VS IDE(cmake 3.8.0 or later required) Set working
    # directory to ${BUILD_TARGET} git repo root.
    if(CMAKE_VERSION VERSION_GREATER 3.8.0)
      set_target_properties(${SAMPLE_TARGET}
              PROPERTIES VS_DEBUGGER_WORKING_DIRECTORY
              "${CMAKE_CURRENT_SOURCE_DIR}")
    endif()
  endif(WIN32)

endif (TINYEXR_BUILD_SAMPLE)

set(GENERATED_DIR "${CMAKE_CURRENT_BINARY_DIR}/generated")
set(VERSION_CONFIG "${GENERATED_DIR}/${PROJECT_NAME}ConfigVersion.cmake")
set(PROJECT_CONFIG "${GENERATED_DIR}/${PROJECT_NAME}Config.cmake")

set(CONFIG_INSTALL_DIR "${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}")

set(TARGETS_EXPORT_NAME "${PROJECT_NAME}Targets")
set(NAMESPACE "${PROJECT_NAME}::")

include(CMakePackageConfigHelpers)
write_basic_package_version_file(
    "${VERSION_CONFIG}" COMPATIBILITY ExactVersion
)

configure_package_config_file(
    "cmake/Config.cmake.in"
    "${PROJECT_CONFIG}"
    INSTALL_DESTINATION "${CONFIG_INSTALL_DIR}"
)

install(
    TARGETS tinyexr
    EXPORT "${TARGETS_EXPORT_NAME}"
    INCLUDES DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}"
    RUNTIME DESTINATION "${CMAKE_INSTALL_BINDIR}"
    LIBRARY DESTINATION "${CMAKE_INSTALL_LIBDIR}"
    ARCHIVE DESTINATION "${CMAKE_INSTALL_LIBDIR}"
)

install(
    FILES ${TINYEXR_HEADERS}
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/${PROJECT_NAME}
)

install(
    FILES "${PROJECT_CONFIG}" "${VERSION_CONFIG}"
    DESTINATION "${CONFIG_INSTALL_DIR}"
)

install(
    EXPORT "${TARGETS_EXPORT_NAME}"
    NAMESPACE "${NAMESPACE}"
    DESTINATION "${CONFIG_INSTALL_DIR}"
)

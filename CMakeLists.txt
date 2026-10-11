cmake_minimum_required(VERSION 3.7)

project(yamlcpp_vendor)

find_package(ament_cmake REQUIRED)

# Lyrical's own yaml_cpp_vendor is a thin shim that only ships Findyaml-cpp.cmake
# and expects a SYSTEM yaml-cpp (Ubuntu libyaml-cpp-dev). On VxWorks there is no
# system package, so this vendor BUILDS yaml-cpp and installs its cmake config
# into the prefix; the shim's find_package(yaml-cpp CONFIG) then resolves.
# Version 0.8.0 matches Ubuntu 26.04 (lyrical target) + the shim's pkg yaml-cpp-0.8.

option(FORCE_BUILD_VENDOR_PKG "Build yaml-cpp from source" OFF)

if(NOT FORCE_BUILD_VENDOR_PKG)
  find_package(yaml-cpp 0.8 QUIET CONFIG)
endif()

macro(build_yamlcpp)
  set(extra_cmake_args)
  if(DEFINED CMAKE_BUILD_TYPE)
    list(APPEND extra_cmake_args -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE})
  endif()
  if(DEFINED BUILD_SHARED_LIBS)
    list(APPEND extra_cmake_args -DBUILD_SHARED_LIBS=${BUILD_SHARED_LIBS})
  else()
    list(APPEND extra_cmake_args -DBUILD_SHARED_LIBS=ON)
  endif()
  if(DEFINED CMAKE_TOOLCHAIN_FILE)
    list(APPEND extra_cmake_args "-DCMAKE_TOOLCHAIN_FILE=${CMAKE_TOOLCHAIN_FILE}")
  endif()
  list(APPEND extra_cmake_args "-DCMAKE_CXX_FLAGS=${CMAKE_CXX_FLAGS}")
  list(APPEND extra_cmake_args -DYAML_CPP_BUILD_TESTS=OFF -DYAML_CPP_BUILD_TOOLS=OFF)

  include(ExternalProject)

  externalproject_add(yamlcpp_external
    GIT_REPOSITORY https://github.com/jbeder/yaml-cpp.git
    GIT_TAG 0.8.0
    GIT_CONFIG advice.detachedHead=false
    UPDATE_COMMAND ""
    TIMEOUT 600
    CMAKE_ARGS
      -DCMAKE_INSTALL_PREFIX=${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}_install
      ${extra_cmake_args}
  )

  install(
    DIRECTORY
      ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}_install/
    DESTINATION
      ${CMAKE_INSTALL_PREFIX}
    USE_SOURCE_PERMISSIONS
  )
endmacro()

if(NOT yaml-cpp_FOUND)
  message(STATUS "Did not find yaml-cpp 0.8, building from source")
  build_yamlcpp()
endif()

ament_package()

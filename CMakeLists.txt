cmake_minimum_required(VERSION 3.0)
project(mujoco_cmake VERSION 3.2.2)

set(MJ_VER ${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}.${PROJECT_VERSION_PATCH})
set(MJ_URL https://github.com/google-deepmind/mujoco/releases/download/${MJ_VER}/mujoco-${MJ_VER}-linux-x86_64.tar.gz)

# Download the file (if it doesn't exist already)
set(MJ_FILE ${CMAKE_BINARY_DIR}/mujoco-${MJ_VER}-linux-x86_64.tar.gz)
if(NOT EXISTS ${MJ_FILE})
    file(DOWNLOAD ${MJ_URL} ${MJ_FILE} SHOW_PROGRESS)
endif()

# taken from https://github.com/google-deepmind/mujoco/releases/tag/ (ensures download integrity)
set(MJ_SHA256 "2802ee237d3c9e187d9c8bf83188d1ac6c3a92460343e26fa23d41fcd8425490") 

# Use the computed hash in ExternalProject_Add
include(ExternalProject)
ExternalProject_Add(
    mujoco
    PREFIX "mujoco"
    URL ${MJ_URL}
    URL_HASH SHA256=${MJ_SHA256}
    CONFIGURE_COMMAND ""
    BUILD_COMMAND ""
    INSTALL_COMMAND ""
)

set(MUJOCO_SRC_DIR ${CMAKE_CURRENT_BINARY_DIR}/mujoco/src/mujoco)

add_custom_command(
    COMMAND cp ${MUJOCO_SRC_DIR}/bin/compile ${CMAKE_CURRENT_BINARY_DIR}/mujoco_compile
    #OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/mujoco_compile
    TARGET mujoco
    COMMENT "renaming compile to mujoco_compile"
)

# configure and install cmake
configure_file(mujoco_cmakeConfig.cmake.in
    ${CMAKE_CURRENT_BINARY_DIR}/mujoco_cmakeConfig.cmake
    @ONLY)

install(
    FILES
    ${CMAKE_CURRENT_BINARY_DIR}/mujoco_cmakeConfig.cmake
    DESTINATION
    lib/cmake/${PROJECT_NAME}
    )

# install headers
install(
    DIRECTORY
    ${MUJOCO_SRC_DIR}/include/
    DESTINATION
    include/mujoco)

# install binaries (mujoco_compile + plugins)
install(
    PROGRAMS
    ${CMAKE_CURRENT_BINARY_DIR}/mujoco_compile
    DESTINATION
    bin
    )

install(
    DIRECTORY
    ${MUJOCO_SRC_DIR}/bin/mujoco_plugin
    DESTINATION
    bin
    FILES_MATCHING PATTERN *.so*)


install(
    FILES
    ${MUJOCO_SRC_DIR}/lib/libmujoco.so.${MJ_VER}
    DESTINATION
    lib)

cmake_minimum_required(VERSION 3.16)
project(is_flutter_vendor)

find_package(ament_cmake REQUIRED)

include(ExternalProject)

set(FLUTTER_INSTALL_PREFIX "${CMAKE_INSTALL_PREFIX}/opt/flutter")

ExternalProject_Add(flutter_external
  GIT_REPOSITORY    https://github.com/flutter/flutter.git
  GIT_TAG           stable
  GIT_SHALLOW       TRUE
  SOURCE_DIR        "${CMAKE_CURRENT_BINARY_DIR}/flutter-src"
  CONFIGURE_COMMAND ""
  BUILD_COMMAND     ""
  INSTALL_COMMAND   ${CMAKE_COMMAND} -E copy_directory "${CMAKE_CURRENT_BINARY_DIR}/flutter-src" "${FLUTTER_INSTALL_PREFIX}"

  LOG_DOWNLOAD ON
  LOG_UPDATE ON
  LOG_CONFIGURE ON
  LOG_BUILD ON
  LOG_INSTALL ON
)

set(EXTRAS_FILE "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}-extras.cmake")
file(WRITE ${EXTRAS_FILE}
  "set(FLUTTER_SDK_DIR \"${FLUTTER_INSTALL_PREFIX}\")\n"
  "set(FLUTTER_EXECUTABLE \"${FLUTTER_INSTALL_PREFIX}/bin/flutter\")\n"
)

ament_package(
  CONFIG_EXTRAS "${EXTRAS_FILE}"
)

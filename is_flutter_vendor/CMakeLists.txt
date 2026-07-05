cmake_minimum_required(VERSION 3.16)
project(is_flutter_vendor)

find_package(ament_cmake REQUIRED)

# For Ubuntu Jammy, is not in rosdep
execute_process(
  COMMAND dpkg -s libstdc++-12-dev
  RESULT_VARIABLE DPKG_RESULT
  OUTPUT_QUIET
  ERROR_QUIET
)

if(NOT DPKG_RESULT EQUAL 0)
  message(FATAL_ERROR "Dependency libstdc++-12-dev is not installed.")
endif()

include(ExternalProject)

set(FLUTTER_VERSION "3.44.4")
set(FLUTTER_INSTALL_PREFIX "${CMAKE_INSTALL_PREFIX}/opt/flutter")

ExternalProject_Add(flutter_external
  URL               https://storage.googleapis.com/flutter_infra_release/releases/stable/linux/flutter_linux_${FLUTTER_VERSION}-stable.tar.xz
  SOURCE_DIR        "${CMAKE_CURRENT_BINARY_DIR}/flutter-src"
  CONFIGURE_COMMAND ""
  BUILD_COMMAND     "${CMAKE_CURRENT_BINARY_DIR}/flutter-src/bin/flutter" precache --linux
  INSTALL_COMMAND   ${CMAKE_COMMAND} -E copy_directory "${CMAKE_CURRENT_BINARY_DIR}/flutter-src" "${FLUTTER_INSTALL_PREFIX}"
  LOG_DOWNLOAD      ON
  LOG_UPDATE        ON
  LOG_CONFIGURE     ON
  LOG_BUILD         ON
  LOG_INSTALL       ON
)

set(EXTRAS_FILE "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}-extras.cmake")
file(WRITE ${EXTRAS_FILE}
  "set(FLUTTER_SDK_DIR \"${FLUTTER_INSTALL_PREFIX}\")\n"
  "set(FLUTTER_EXECUTABLE \"${FLUTTER_INSTALL_PREFIX}/bin/flutter\")\n"
  "\n"
  "function(ament_flutter_build_app APP_DIR)\n"
  "  set(CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT TRUE)\n"
  "  get_filename_component(APP_EXECUTABLE_NAME \"\${APP_DIR}\" NAME)\n"
  "  add_custom_command(\n"
  "    OUTPUT \"\${APP_DIR}/pubspec.lock\"\n"
  "    COMMAND \${FLUTTER_EXECUTABLE} pub get\n"
  "    WORKING_DIRECTORY \${APP_DIR}\n"
  "  )\n"
  "  add_custom_command(\n"
  "    OUTPUT \"\${APP_DIR}/build/linux/x64/release/bundle/\${APP_EXECUTABLE_NAME}\"\n"
  "    COMMAND \${FLUTTER_EXECUTABLE} config --enable-linux-desktop\n"
  "    COMMAND \${FLUTTER_EXECUTABLE} build linux --release\n"
  "    DEPENDS \"\${APP_DIR}/pubspec.lock\"\n"
  "    WORKING_DIRECTORY \${APP_DIR}\n"
  "  )\n"
  "  add_custom_target(\${APP_EXECUTABLE_NAME}_target ALL\n"
  "    DEPENDS \"\${APP_DIR}/build/linux/x64/release/bundle/\${APP_EXECUTABLE_NAME}\"\n"
  "  )\n"
  "  install(DIRECTORY \"\${APP_DIR}/build/linux/x64/release/bundle/\"\n"
  "    DESTINATION \"lib/\${PROJECT_NAME}\"\n"
  "    USE_SOURCE_PERMISSIONS\n"
  "  )\n"
  "endfunction()\n"
)

ament_package(
  CONFIG_EXTRAS "${EXTRAS_FILE}"
)
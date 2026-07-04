cmake_minimum_required(VERSION 3.16)
project(isaac_app)

find_package(ament_cmake REQUIRED)
find_package(is_flutter_vendor REQUIRED)

set(FLUTTER_APP_DIR "${CMAKE_CURRENT_SOURCE_DIR}/flutter_ui")

add_custom_command(
  OUTPUT "${FLUTTER_APP_DIR}/pubspec.lock"
  COMMAND ${FLUTTER_EXECUTABLE} pub get 
  WORKING_DIRECTORY ${FLUTTER_APP_DIR}
  COMMENT "Getting Flutter dependencies..."
)

add_custom_command(
  OUTPUT "${FLUTTER_APP_DIR}/build/linux/x64/release/bundle/flutter_ui"
  COMMAND ${FLUTTER_EXECUTABLE} config --enable-linux-desktop
  COMMAND ${FLUTTER_EXECUTABLE} build linux --release
  DEPENDS "${FLUTTER_APP_DIR}/pubspec.lock"
  WORKING_DIRECTORY ${FLUTTER_APP_DIR}
  COMMENT "Building Flutter Linux app..."
)

add_custom_target(build_flutter_app ALL
  DEPENDS "${FLUTTER_APP_DIR}/build/linux/x64/release/bundle/flutter_ui"
)

install(DIRECTORY "${FLUTTER_APP_DIR}/build/linux/x64/release/bundle/"
  DESTINATION "lib/${PROJECT_NAME}"
  USE_SOURCE_PERMISSIONS
)


ament_package()
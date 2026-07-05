cmake_minimum_required(VERSION 3.16)
project(isaac_app)

find_package(ament_cmake REQUIRED)
find_package(is_flutter_vendor REQUIRED)

ament_flutter_build_app("${CMAKE_CURRENT_SOURCE_DIR}/flutter_ui")

ament_package()
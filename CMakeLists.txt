cmake_minimum_required(VERSION 3.0)

project(websocketpp VERSION 0.7.0)

set(include_install_dir "include")
set(config_install_dir "lib/cmake/${PROJECT_NAME}")

add_library(websocketpp INTERFACE)
target_include_directories(websocketpp INTERFACE
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/websocketpp>
  $<INSTALL_INTERFACE:${include_install_dir}>)

set(config_export_name "${PROJECT_NAME}Config")
set(targets_export_name "${PROJECT_NAME}Targets")
set(project_config "${CMAKE_CURRENT_BINARY_DIR}/${config_export_name}.cmake")
set(version_config "${CMAKE_CURRENT_BINARY_DIR}/${config_export_name}Version.cmake")
set(namespace "${PROJECT_NAME}::")

include(CMakePackageConfigHelpers)
configure_package_config_file( # Uses target_exports_name                                                                                                                                 
    "cmake/hunter.Config.cmake.in"
    "${project_config}"
    INSTALL_DESTINATION "${config_install_dir}"
)
write_basic_package_version_file(
  "${version_config}" COMPATIBILITY SameMajorVersion)

install(
  TARGETS websocketpp
  EXPORT "${targets_export_name}"
  INCLUDES DESTINATION "${include_install_dir}")

install(
  DIRECTORY "websocketpp"
  DESTINATION "${include_install_dir}"
  PATTERN CMakeLists.txt EXCLUDE)

install(
  EXPORT "${targets_export_name}"
  NAMESPACE "${namespace}"
  DESTINATION "${config_install_dir}")
install(
  FILES "${project_config}" "${version_config}"
  DESTINATION "${config_install_dir}")



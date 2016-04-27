cmake_minimum_required(VERSION 3.0)

include("cmake/HunterGate.cmake")
HunterGate(
  URL "https://github.com/ruslo/hunter/archive/v0.14.13.tar.gz"
  SHA1 "95a031bbab876c3c049b3cd4c65b7a2981dad202"
)

option(ENABLE_CPP11 "Build websocketpp with CPP11 features enabled." TRUE)

project(websocketpp VERSION 0.7.0)

set(include_install_dir "include")
set(config_install_dir "lib/cmake/${PROJECT_NAME}")

hunter_add_package(ZLIB)
hunter_add_package(OpenSSL)
hunter_add_package(Boost COMPONENTS system random thread)

find_package(ZLIB CONFIG REQUIRED)
find_package(OpenSSL REQUIRED)
find_package(Boost CONFIG COMPONENTS system random thread)

add_library(websocketpp INTERFACE)
target_include_directories(websocketpp INTERFACE
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/websocketpp>
  $<INSTALL_INTERFACE:${include_install_dir}>)
target_link_libraries(websocketpp INTERFACE
  ${CMAKE_DL_LIBS}
  ZLIB::zlib
  OpenSSL::SSL
  OpenSSL::Crypto
  Boost::system
  Boost::random
  Boost::thread)

# VC9 and C++11 reasoning
if (ENABLE_CPP11 AND MSVC AND MSVC90)
  message("* Detected Visual Studio 9 2008, disabling C++11 support.")
  set (ENABLE_CPP11 FALSE)
endif ()

# C++11 defines
if (ENABLE_CPP11)
  if (MSVC)
    target_compile_definitions (websocketpp INTERFACE _WEBSOCKETPP_CPP11_FUNCTIONAL_)
    target_compile_definitions (websocketpp INTERFACE _WEBSOCKETPP_CPP11_SYSTEM_ERROR_)
    target_compile_definitions (websocketpp INTERFACE _WEBSOCKETPP_CPP11_RANDOM_DEVICE_)
    target_compile_definitions (websocketpp INTERFACE _WEBSOCKETPP_CPP11_MEMORY_)
  else()
    target_compile_definitions (websocketpp INTERFACE _WEBSOCKETPP_CPP11_STL_)
  endif()
endif ()


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



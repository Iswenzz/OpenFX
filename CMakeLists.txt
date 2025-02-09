cmake_minimum_required(VERSION 3.16.0)
project(openfx VERSION 1.4.0 LANGUAGES CXX)
set_property(GLOBAL PROPERTY USE_FOLDERS ON)
set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_STATIC_LIBRARY_PREFIX "lib")
add_definitions(-D_HAS_AUTO_PTR_ETC)
if(APPLE)
   set(CMAKE_OSX_ARCHITECTURES "x86_64;arm64")
endif()
if(WIN32)
	add_definitions(-DWINDOWS -DNOMINMAX)
endif()

option(BUILD_EXAMPLE_PLUGINS "Build example plugins" OFF)
option(OFX_SUPPORTS_OPENGLRENDER "Build with support for GPU rendering (OpenGL/CUDA/Metal/OpenCL)" ON)

# Flags
if(!MSVC)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wextra -Wno-deprecated -Wno-deprecated-declarations")
endif()

list(APPEND CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake ${CMAKE_BINARY_DIR} )

# Dependencies
find_package(expat CONFIG REQUIRED)
find_package(OpenGL REQUIRED)

# Macros
set(OFX_SUPPORT_SYMBOLS_DIR ${PROJECT_SOURCE_DIR}/symbols)
include(OpenFX)
set_property(GLOBAL PROPERTY OFX_PROJECT_SOURCE_DIR "${PROJECT_SOURCE_DIR}")

# C Headers
set(OFX_HEADER_DIR "${CMAKE_CURRENT_SOURCE_DIR}/include")
file(GLOB_RECURSE OFX_HEADER_FILES "${OFX_HEADER_DIR}/*.h")

# Defines
if(OFX_SUPPORTS_OPENGLRENDER)
  add_compile_definitions(OFX_SUPPORTS_OPENGLRENDER)
endif()

# Build
add_subdirectory(HostSupport)
add_subdirectory(Support)
if(BUILD_EXAMPLE_PLUGINS)
  add_subdirectory(Examples)
endif()

# Install
install(TARGETS OfxSupport OfxHost
	EXPORT OpenFXTargets
	RUNTIME DESTINATION bin
	LIBRARY DESTINATION lib
	ARCHIVE DESTINATION lib)

install(DIRECTORY ${OFX_SUPPORT_SYMBOLS_DIR}/ DESTINATION symbols)

# Package
include(CMakePackageConfigHelpers)

export(EXPORT OpenFXTargets
	FILE "${CMAKE_CURRENT_BINARY_DIR}/OpenFXTargets.cmake"
	NAMESPACE OpenFX::)

configure_package_config_file(cmake/OpenFXConfig.cmake.in
	"${CMAKE_CURRENT_BINARY_DIR}/OpenFXConfig.cmake"
	INSTALL_DESTINATION lib/cmake/OpenFX)

install(EXPORT OpenFXTargets
	FILE OpenFXTargets.cmake
	NAMESPACE OpenFX::
	DESTINATION lib/cmake/OpenFX)

install(FILES "${CMAKE_CURRENT_BINARY_DIR}/OpenFXConfig.cmake"
	DESTINATION lib/cmake/OpenFX)

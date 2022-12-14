cmake_minimum_required(VERSION 3.20)
project(WiVRn VERSION 1.0 LANGUAGES C CXX)

set(CMAKE_POLICY_DEFAULT_CMP0077 NEW)

if(${CMAKE_VERSION} VERSION_GREATER_EQUAL 3.24)
    cmake_policy(SET CMP0135 NEW)
endif()

set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

include(FetchContent)
include(ExternalProject)

option(WIVRN_USE_SYSTEM_FREETYPE "Use system FreeType" ON)
option(WIVRN_USE_ANDROID_VALIDATION_LAYER "Download the android version of the Vulkan validation layer" OFF)
option(WIVRN_BUILD_CLIENT "Build WiVRn client" ON)
option(WIVRN_BUILD_SERVER "Build WiVRn server" ON)

option(ENABLE_COLOURED_OUTPUT "Always produce ANSI-coloured output (GNU/Clang only)." ON)

if(ENABLE_COLOURED_OUTPUT)
	if("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
		add_compile_options(-fdiagnostics-color=always)
	elseif("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang")
		add_compile_options(-fcolor-diagnostics)
	endif()
endif()

find_package(PkgConfig REQUIRED)
find_package(Vulkan REQUIRED)
pkg_check_modules(systemd IMPORTED_TARGET systemd)

if (WIVRN_BUILD_SERVER)
    find_package(CUDAToolkit)
    pkg_check_modules(LIBAV IMPORTED_TARGET libavcodec libavutil libswscale libavfilter)
    pkg_check_modules(X264  IMPORTED_TARGET x264)

    if(CUDAToolkit_FOUND)
        set(WIVRN_HAVE_CUDA ON)
    endif()

    if(LIBAV_FOUND)
        set(WIVRN_HAVE_FFMPEG ON)
    endif()

    if(X264_FOUND)
        set(WIVRN_HAVE_X264 ON)
    endif()

    if (NOT CUDAToolkit_FOUND AND NOT LIBAV_FOUND AND NOT X264_FOUND)
        message(FATAL_ERROR "No encoder found")
    endif()

    pkg_check_modules(AVAHI IMPORTED_TARGET avahi-client)
    pkg_check_modules(libpulse IMPORTED_TARGET libpulse)
    find_package(Eigen3 REQUIRED)
endif()

if (WIVRN_BUILD_CLIENT AND NOT ANDROID)
    pkg_check_modules(LIBAV IMPORTED_TARGET libavcodec libavutil libswscale libavfilter)
endif()

set(CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake)

if(ANDROID)
    include(AndroidNdkModules)
    android_ndk_import_module_native_app_glue()
endif()

FetchContent_Declare(spdlog   URL https://github.com/gabime/spdlog/archive/refs/tags/v1.11.0.tar.gz)
FetchContent_Declare(glm      URL https://github.com/g-truc/glm/archive/refs/tags/0.9.9.8.tar.gz)
FetchContent_Declare(tinygltf URL https://github.com/syoyo/tinygltf/archive/refs/tags/v2.6.3.tar.gz)
FetchContent_Declare(boostpfr URL https://github.com/boostorg/pfr/archive/refs/tags/2.0.3.tar.gz)
FetchContent_Declare(monado
    GIT_REPOSITORY   https://gitlab.freedesktop.org/xytovl/monado
    GIT_TAG          3046a5eee4105c4d1c07bc466d6e3a0bb749df42
    )


add_subdirectory(external)
add_subdirectory(common)

if (WIVRN_BUILD_CLIENT)
    add_subdirectory(client)
endif()

if (WIVRN_BUILD_SERVER)
    find_package(nlohmann_json REQUIRED)
    add_subdirectory(server)
endif()

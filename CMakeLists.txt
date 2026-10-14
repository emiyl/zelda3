cmake_minimum_required(VERSION 3.16)

option(ZELDA3_IOS "Build an iOS app bundle" OFF)
if(ZELDA3_IOS)
  set(CMAKE_SYSTEM_NAME iOS)
endif()
if(CMAKE_SYSTEM_NAME STREQUAL "iOS")
  set(CMAKE_OSX_DEPLOYMENT_TARGET "15.0" CACHE STRING "Minimum iOS version")
endif()

project(zelda3 LANGUAGES C)

if(CMAKE_SYSTEM_NAME STREQUAL "iOS")
  set(ZELDA3_IOS ON)
endif()

set(SDL_SHARED OFF CACHE BOOL "Build shared SDL2" FORCE)
set(SDL_STATIC ON CACHE BOOL "Build static SDL2" FORCE)
set(SDL_TEST OFF CACHE BOOL "Build SDL2 test library" FORCE)
set(SDL_TESTS OFF CACHE BOOL "Build SDL2 tests" FORCE)
add_subdirectory(third_party/SDL EXCLUDE_FROM_ALL)

file(GLOB ZELDA3_SOURCES CONFIGURE_DEPENDS src/*.c snes/*.c)
add_executable(zelda3
  ${ZELDA3_SOURCES}
  third_party/gl_core/gl_core_3_1.c
  third_party/opus-1.3.1-stripped/opus_decoder_amalgam.c
)
set_target_properties(zelda3 PROPERTIES
  C_STANDARD 11
  C_STANDARD_REQUIRED ON
)
target_include_directories(zelda3 PRIVATE "${CMAKE_CURRENT_SOURCE_DIR}")
target_compile_definitions(zelda3 PRIVATE SYSTEM_VOLUME_MIXER_AVAILABLE=0)

if(TARGET SDL2::SDL2main)
  target_link_libraries(zelda3 PRIVATE SDL2::SDL2main)
endif()
target_link_libraries(zelda3 PRIVATE SDL2::SDL2-static)

if(ZELDA3_IOS)
  if(NOT TARGET SDL2::SDL2main)
    message(FATAL_ERROR "The iOS build requires SDL2main; set SDL2_DISABLE_SDL2MAIN=OFF.")
  endif()
  if(NOT EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/zelda3_assets.dat")
    message(FATAL_ERROR "Generate zelda3_assets.dat with python3 assets/restool.py --extract-from-rom before configuring for iOS.")
  endif()
  set(ZELDA3_RESOURCES
    "${CMAKE_CURRENT_SOURCE_DIR}/zelda3.ini"
    "${CMAKE_CURRENT_SOURCE_DIR}/zelda3_assets.dat"
  )
  target_sources(zelda3 PRIVATE ${ZELDA3_RESOURCES})
  set_source_files_properties(${ZELDA3_RESOURCES} PROPERTIES MACOSX_PACKAGE_LOCATION Resources)
  set_target_properties(zelda3 PROPERTIES
    MACOSX_BUNDLE TRUE
    MACOSX_BUNDLE_INFO_PLIST "${CMAKE_CURRENT_SOURCE_DIR}/src/platform/ios/Info.plist.in"
    MACOSX_BUNDLE_GUI_IDENTIFIER "org.zelda3.zelda3"
    MACOSX_BUNDLE_BUNDLE_NAME "Zelda3"
    MACOSX_BUNDLE_BUNDLE_VERSION "1"
    MACOSX_BUNDLE_SHORT_VERSION_STRING "1.0"
    XCODE_ATTRIBUTE_TARGETED_DEVICE_FAMILY "1,2"
    XCODE_ATTRIBUTE_CODE_SIGN_STYLE "Automatic"
  )
endif()

if(WIN32)
  enable_language(RC)
  target_sources(zelda3 PRIVATE src/platform/win32/zelda3.rc)
endif()

if(MSVC)
  target_compile_definitions(zelda3 PRIVATE _CRT_SECURE_NO_WARNINGS)
endif()

if(NOT WIN32)
  target_link_libraries(zelda3 PRIVATE m)
endif()

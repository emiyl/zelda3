cmake_minimum_required(VERSION 3.16)
project(zelda3 LANGUAGES C)

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

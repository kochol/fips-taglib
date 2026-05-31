#
# fips-taglib  -  fipsified build wrapper for TagLib 2.x
#
# Builds a single static library 'tag' from the upstream sources under taglib/.
# All format modules are enabled, zlib is left off (optional, only used for
# compressed ID3v2 frames), and the two generated config headers are produced
# from the upstream templates. Importers just do fips_deps(tag).
#
cmake_minimum_required(VERSION 3.21)

if (NOT FIPS_IMPORT)
    get_filename_component(FIPS_ROOT_DIR "../fips" ABSOLUTE)
    include("${FIPS_ROOT_DIR}/cmake/fips.cmake")
    set(CMAKE_CXX_STANDARD 17)
    fips_setup()
    fips_project(fips-taglib)
endif()

# enable every format module (these drive taglib_config.h)
set(TAGLIB_WITH_APE       TRUE)
set(TAGLIB_WITH_ASF       TRUE)
set(TAGLIB_WITH_DSF       TRUE)
set(TAGLIB_WITH_MATROSKA  TRUE)
set(TAGLIB_WITH_MOD       TRUE)
set(TAGLIB_WITH_MP4       TRUE)
set(TAGLIB_WITH_RIFF      TRUE)
set(TAGLIB_WITH_SHORTEN   TRUE)
set(TAGLIB_WITH_TRUEAUDIO TRUE)
set(TAGLIB_WITH_VORBIS    TRUE)

# byteswap / ISO _strdup feature detection -> config.h
include(${CMAKE_CURRENT_SOURCE_DIR}/ConfigureChecks.cmake)

# generated config headers (compile-time requirements)
configure_file(
    ${CMAKE_CURRENT_SOURCE_DIR}/config.h.cmake
    ${CMAKE_CURRENT_BINARY_DIR}/config.h)
configure_file(
    ${CMAKE_CURRENT_SOURCE_DIR}/taglib/taglib_config.h.cmake
    ${CMAKE_CURRENT_BINARY_DIR}/taglib_config.h)

# the TagLib static library (all sources under taglib/, recursively)
fips_begin_lib(tag)
    fips_src(taglib GROUP_FOLDERS)
fips_end_lib()

# TagLib sources include each other by bare filename across many sub-folders,
# so every directory holding a header must be on the include path, along with
# the bundled utf8-cpp and the generated config headers.
file(GLOB_RECURSE _taglib_headers ${CMAKE_CURRENT_SOURCE_DIR}/taglib/*.h)
set(_taglib_include_dirs "")
foreach (_h ${_taglib_headers})
    get_filename_component(_d ${_h} DIRECTORY)
    list(APPEND _taglib_include_dirs ${_d})
endforeach()
list(REMOVE_DUPLICATES _taglib_include_dirs)

target_include_directories(tag PUBLIC
    ${_taglib_include_dirs}
    ${CMAKE_CURRENT_SOURCE_DIR}/3rdparty/utfcpp/source
    ${CMAKE_CURRENT_BINARY_DIR})

# static link: TAGLIB_EXPORT expands to nothing (no dllimport/export)
target_compile_definitions(tag PUBLIC TAGLIB_STATIC)

if (FIPS_MSVC)
    target_compile_options(tag PRIVATE /wd4251 /wd4244 /wd4267 /wd4275 /wd4996 /WX-)
endif()

if (NOT FIPS_IMPORT)
    fips_finish()
endif()

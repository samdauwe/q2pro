cmake_minimum_required(VERSION 3.16)

# ---------------------------------------------------------------------------
# Version: read from VERSION file or fall back to git describe
# ---------------------------------------------------------------------------
if(EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/VERSION")
    file(READ "${CMAKE_CURRENT_SOURCE_DIR}/VERSION" Q2PRO_VERSION)
    string(STRIP "${Q2PRO_VERSION}" Q2PRO_VERSION)
else()
    execute_process(
        COMMAND git rev-list --count HEAD
        WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}"
        OUTPUT_VARIABLE _rev OUTPUT_STRIP_TRAILING_WHITESPACE
        ERROR_QUIET RESULT_VARIABLE _git_ok)
    execute_process(
        COMMAND git rev-parse --short HEAD
        WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}"
        OUTPUT_VARIABLE _sha OUTPUT_STRIP_TRAILING_WHITESPACE
        ERROR_QUIET)
    if(_git_ok EQUAL 0 AND _rev AND _sha)
        set(Q2PRO_VERSION "r${_rev}~${_sha}")
    else()
        set(Q2PRO_VERSION "r666~unknown")
    endif()
endif()

# Extract numeric revision for REVISION define
string(REGEX MATCH "r([0-9]+)" _rev_match "${Q2PRO_VERSION}")
if(_rev_match)
    set(Q2PRO_REVISION "${CMAKE_MATCH_1}")
else()
    set(Q2PRO_REVISION 0)
endif()

project(q2pro VERSION 1.0 LANGUAGES C)
message(STATUS "Q2PRO version: ${Q2PRO_VERSION} (revision ${Q2PRO_REVISION})")

set(CMAKE_C_STANDARD 11)
set(CMAKE_C_STANDARD_REQUIRED ON)
set(CMAKE_C_EXTENSIONS ON)

# ---------------------------------------------------------------------------
# Build options
# ---------------------------------------------------------------------------
option(Q2PRO_SYSTEM_WIDE         "Enable system-wide installation"                  ON)
option(Q2PRO_ANTICHEAT_SERVER    "Enable r1ch.net anticheat server interface"        OFF)
option(Q2PRO_AUTO_REPLY          "Make client auto reply to !version chat probes"    ON)
option(Q2PRO_CLIENT_GTV          "Experimental client GTV relay"                     OFF)
option(Q2PRO_CLIENT_UI           "Enable menu-based user interface"                  ON)
option(Q2PRO_DEBUG               "Enable debug code"                                 OFF)
option(Q2PRO_GAME_NEW_API        "Build game using new API for big maps support"     OFF)
option(Q2PRO_ICMP_ERRORS         "Handle ICMP errors on UDP sockets"                 ON)
option(Q2PRO_MD3                 "MD3 models support"                                ON)
option(Q2PRO_MD5                 "MD5 (re-release) models support"                   ON)
option(Q2PRO_MVD_CLIENT          "Enable MVD/GTV client functionality"               ON)
option(Q2PRO_MVD_SERVER          "Enable local MVD recording and MVD/GTV server"     ON)
option(Q2PRO_OPENGL_ES1          "Build OpenGL ES 1 compatible renderer"             OFF)
option(Q2PRO_PACKETDUP_HACK      "Server side packet duplication hack"               OFF)
option(Q2PRO_SAVE_GAMES          "Server side support for savegames"                 ON)
option(Q2PRO_SOFTWARE_SOUND      "Software sound backend"                            ON)
option(Q2PRO_SYSTEM_CONSOLE      "Enhanced system console support"                   ON)
option(Q2PRO_TESTS               "Enable built-in testing code (DANGEROUS)"          OFF)
option(Q2PRO_TGA                 "TGA images support"                                ON)
option(Q2PRO_VARIABLE_FPS        "Support for variable server FPS"                   OFF)
option(Q2PRO_GAME_ABI_HACK       "x86 Linux game ABI compatibility hack"             OFF)

# Dependency toggles: AUTO / ON / OFF
set(Q2PRO_USE_ZLIB    "AUTO" CACHE STRING "zlib support (AUTO/ON/OFF)")
set(Q2PRO_USE_PNG     "AUTO" CACHE STRING "libpng support (AUTO/ON/OFF)")
set(Q2PRO_USE_JPEG    "AUTO" CACHE STRING "libjpeg-turbo support (AUTO/ON/OFF)")
set(Q2PRO_USE_CURL    "AUTO" CACHE STRING "libcurl support (AUTO/ON/OFF)")
set(Q2PRO_USE_SDL2    "AUTO" CACHE STRING "SDL2 support (AUTO/ON/OFF)")
set(Q2PRO_USE_OPENAL  "AUTO" CACHE STRING "OpenAL support (AUTO/ON/OFF)")
set(Q2PRO_USE_AVCODEC "AUTO" CACHE STRING "FFmpeg/libavcodec support (AUTO/ON/OFF)")
set(Q2PRO_USE_X11     "AUTO" CACHE STRING "X11 video driver (AUTO/ON/OFF)")
set(Q2PRO_USE_WAYLAND "AUTO" CACHE STRING "Wayland video driver (AUTO/ON/OFF)")

# String options
set(Q2PRO_BASE_GAME     "baseq2"                    CACHE STRING "Base game directory name")
set(Q2PRO_DEFAULT_GAME  ""                           CACHE STRING "Default value for game console variable")
set(Q2PRO_HOMEDIR       "~/.q2pro"                   CACHE STRING "Default homedir console variable")
set(Q2PRO_VID_GEOMETRY  "640x480"                    CACHE STRING "Default window geometry")
set(Q2PRO_VID_MODELIST  "640x480 800x600 1024x768"   CACHE STRING "Default video mode list")

# ---------------------------------------------------------------------------
# Platform detection
# ---------------------------------------------------------------------------
include(CheckCCompilerFlag)
include(CheckIncludeFile)
include(CheckSymbolExists)
include(CheckFunctionExists)
include(CMakeDependentOption)
include(GNUInstallDirs)
include(FindPkgConfig)

set(Q2PRO_WIN32 OFF)
if(WIN32)
    set(Q2PRO_WIN32 ON)
endif()

set(Q2PRO_X86 OFF)
if(CMAKE_SYSTEM_PROCESSOR MATCHES "^(i[3-6]86|x86)$")
    set(Q2PRO_X86 ON)
endif()

# CPU string for game library naming
set(Q2PRO_CPU "${CMAKE_SYSTEM_PROCESSOR}")
if(Q2PRO_CPU STREQUAL "x86_64" OR Q2PRO_CPU STREQUAL "AMD64")
    if(Q2PRO_WIN32)
        set(Q2PRO_CPU "x86_64")
    else()
        set(Q2PRO_CPU "x86_64")
    endif()
elseif(Q2PRO_CPU MATCHES "^i[3-6]86$")
    if(Q2PRO_WIN32)
        set(Q2PRO_CPU "x86")
    else()
        set(Q2PRO_CPU "i386")
    endif()
elseif(Q2PRO_CPU STREQUAL "aarch64")
    set(Q2PRO_CPU "arm64")
endif()

# Build string
if(Q2PRO_WIN32)
    if(Q2PRO_CPU MATCHES "64")
        set(Q2PRO_BUILDSTRING "Win64")
    else()
        set(Q2PRO_BUILDSTRING "Win32")
    endif()
elseif(APPLE)
    set(Q2PRO_BUILDSTRING "Darwin")
elseif(CMAKE_SYSTEM_NAME MATCHES ".*[Bb][Ss][Dd]$")
    string(LENGTH "${CMAKE_SYSTEM_NAME}" _len)
    math(EXPR _prefix_len "${_len} - 3")
    string(SUBSTRING "${CMAKE_SYSTEM_NAME}" 0 ${_prefix_len} _prefix)
    string(SUBSTRING "${CMAKE_SYSTEM_NAME}" ${_prefix_len} 3 _suffix)
    string(TOUPPER "${_suffix}" _suffix)
    set(Q2PRO_BUILDSTRING "${_prefix}${_suffix}")
else()
    string(SUBSTRING "${CMAKE_SYSTEM_NAME}" 0 1 _first)
    string(TOUPPER "${_first}" _first)
    string(SUBSTRING "${CMAKE_SYSTEM_NAME}" 1 -1 _rest)
    set(Q2PRO_BUILDSTRING "${_first}${_rest}")
endif()

# ---------------------------------------------------------------------------
# Compiler flags
# ---------------------------------------------------------------------------
if(NOT MSVC)
    if(Q2PRO_X86)
        add_compile_options(-msse2 -mfpmath=sse)
    endif()

    foreach(_flag
        -Werror=vla
        -Wformat-security
        -Wno-microsoft-anon-tag
        -Wpointer-arith
        -Wstrict-prototypes
        -fms-extensions
        -fno-math-errno
        -fno-trapping-math
        -fsigned-char)
        string(MAKE_C_IDENTIFIER "HAS${_flag}" _var)
        check_c_compiler_flag("${_flag}" ${_var})
        if(${_var})
            add_compile_options("${_flag}")
        endif()
    endforeach()

    check_c_compiler_flag("-Wmissing-prototypes" HAS_WMISSING_PROTOTYPES)
else()
    add_compile_options(/wd4146 /wd4244 /wd4305)
endif()

# Common definitions
add_compile_definitions(HAVE_CONFIG_H)
if(Q2PRO_WIN32)
    add_compile_definitions(_USE_MATH_DEFINES)
else()
    add_compile_definitions(_GNU_SOURCE)
endif()

# ---------------------------------------------------------------------------
# Install paths
# ---------------------------------------------------------------------------
if(NOT Q2PRO_WIN32 AND Q2PRO_SYSTEM_WIDE)
    set(Q2PRO_DATADIR "${CMAKE_INSTALL_FULL_DATADIR}/${PROJECT_NAME}")
    set(Q2PRO_LIBDIR  "${CMAKE_INSTALL_FULL_LIBDIR}/${PROJECT_NAME}")
    set(Q2PRO_HOMEDIR_CFG "${Q2PRO_HOMEDIR}")
    set(Q2PRO_HAS_SYSTEM_PATHS ON)
else()
    set(Q2PRO_DATADIR ".")
    set(Q2PRO_LIBDIR  ".")
    set(Q2PRO_HOMEDIR_CFG "")
    set(Q2PRO_HAS_SYSTEM_PATHS OFF)
endif()

# ---------------------------------------------------------------------------
# Source tree
# ---------------------------------------------------------------------------
add_subdirectory(src)

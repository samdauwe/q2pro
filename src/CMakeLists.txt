# ===========================================================================
# Q2PRO – src/CMakeLists.txt
# ===========================================================================

# ---------------------------------------------------------------------------
# Source lists
# ---------------------------------------------------------------------------
set(COMMON_SRC
    common/bsp.c
    common/cmd.c
    common/cmodel.c
    common/common.c
    common/crc.c
    common/cvar.c
    common/error.c
    common/field.c
    common/fifo.c
    common/files.c
    common/hash_map.c
    common/math.c
    common/mdfour.c
    common/msg.c
    common/net/chan.c
    common/net/net.c
    common/pmove/common.c
    common/pmove/new.c
    common/pmove/old.c
    common/prompt.c
    common/sizebuf.c
    common/utils.c
    common/zone.c
    shared/shared.c
)

set(CLIENT_SRC
    client/ascii.c
    client/console.c
    client/demo.c
    client/download.c
    client/effects.c
    client/entities.c
    client/input.c
    client/keys.c
    client/locs.c
    client/main.c
    client/newfx.c
    client/parse.c
    client/precache.c
    client/predict.c
    client/refresh.c
    client/screen.c
    client/sound/main.c
    client/sound/mem.c
    client/tent.c
    client/view.c
    common/async.c
    server/commands.c
    server/entities.c
    server/game.c
    server/init.c
    server/main.c
    server/send.c
    server/user.c
    server/world.c
    shared/m_flash.c
)

set(UI_SRC
    client/ui/demos.c
    client/ui/menu.c
    client/ui/playerconfig.c
    client/ui/playermodels.c
    client/ui/script.c
    client/ui/servers.c
    client/ui/ui.c
)

set(REFRESH_SRC
    refresh/images.c
)

set(SRC_GL
    refresh/gl/draw.c
    refresh/gl/hq2x.c
    refresh/gl/legacy.c
    refresh/gl/main.c
    refresh/gl/mesh.c
    refresh/gl/models.c
    refresh/gl/qgl.c
    refresh/gl/shader.c
    refresh/gl/sky.c
    refresh/gl/state.c
    refresh/gl/surf.c
    refresh/gl/tess.c
    refresh/gl/texture.c
    refresh/gl/world.c
)

set(SERVER_SRC
    client/null.c
    server/commands.c
    server/entities.c
    server/game.c
    server/init.c
    server/main.c
    server/send.c
    server/user.c
    server/world.c
)

set(GAME_SRC
    game/g_ai.c
    game/g_chase.c
    game/g_cmds.c
    game/g_combat.c
    game/g_func.c
    game/g_items.c
    game/g_main.c
    game/g_misc.c
    game/g_monster.c
    game/g_phys.c
    game/g_ptrs.c
    game/g_save.c
    game/g_spawn.c
    game/g_svcmds.c
    game/g_target.c
    game/g_trigger.c
    game/g_turret.c
    game/g_utils.c
    game/g_weapon.c
    game/m_actor.c
    game/m_berserk.c
    game/m_boss2.c
    game/m_boss31.c
    game/m_boss32.c
    game/m_boss3.c
    game/m_brain.c
    game/m_chick.c
    game/m_flipper.c
    game/m_float.c
    game/m_flyer.c
    game/m_gladiator.c
    game/m_gunner.c
    game/m_hover.c
    game/m_infantry.c
    game/m_insane.c
    game/m_medic.c
    game/m_move.c
    game/m_mutant.c
    game/m_parasite.c
    game/m_soldier.c
    game/m_supertank.c
    game/m_tank.c
    game/p_client.c
    game/p_hud.c
    game/p_trail.c
    game/p_view.c
    game/p_weapon.c
    shared/m_flash.c
    shared/shared.c
)

# ---------------------------------------------------------------------------
# Helper: try to find a dependency (AUTO/ON/OFF logic)
# ---------------------------------------------------------------------------
macro(q2pro_find_dependency VAR PKG)
    set(_mode "${Q2PRO_USE_${VAR}}")
    if(_mode STREQUAL "ON")
        find_package(${PKG} REQUIRED ${ARGN})
    elseif(_mode STREQUAL "AUTO")
        find_package(${PKG} QUIET ${ARGN})
    endif()
    # OFF => skip entirely
endmacro()

macro(q2pro_pkg_check VAR PKG)
    set(_mode "${Q2PRO_USE_${VAR}}")
    if(_mode STREQUAL "ON")
        pkg_check_modules(${PKG} REQUIRED IMPORTED_TARGET ${ARGN})
    elseif(_mode STREQUAL "AUTO")
        pkg_check_modules(${PKG} QUIET IMPORTED_TARGET ${ARGN})
    endif()
endmacro()

# ---------------------------------------------------------------------------
# Dependencies
# ---------------------------------------------------------------------------
# zlib
q2pro_find_dependency(ZLIB ZLIB)
set(HAVE_ZLIB OFF)
if(ZLIB_FOUND)
    set(HAVE_ZLIB ON)
endif()

# libpng
q2pro_find_dependency(PNG PNG)
set(HAVE_PNG OFF)
if(PNG_FOUND)
    set(HAVE_PNG ON)
endif()

# libjpeg
set(HAVE_JPG OFF)
set(_jpeg_mode "${Q2PRO_USE_JPEG}")
if(NOT _jpeg_mode STREQUAL "OFF")
    if(_jpeg_mode STREQUAL "ON")
        find_package(JPEG REQUIRED)
    else()
        find_package(JPEG QUIET)
    endif()
    if(JPEG_FOUND)
        # Check for JCS_EXT_RGBA (libjpeg-turbo)
        include(CheckCSourceCompiles)
        set(CMAKE_REQUIRED_INCLUDES ${JPEG_INCLUDE_DIRS})
        set(CMAKE_REQUIRED_LIBRARIES ${JPEG_LIBRARIES})
        check_c_source_compiles("
            #include <stdio.h>
            #include <jpeglib.h>
            int main(void) { int x = JCS_EXT_RGBA; return x; }
        " HAVE_JCS_EXT_RGBA)
        unset(CMAKE_REQUIRED_INCLUDES)
        unset(CMAKE_REQUIRED_LIBRARIES)
        if(HAVE_JCS_EXT_RGBA)
            set(HAVE_JPG ON)
        elseif(_jpeg_mode STREQUAL "ON")
            message(FATAL_ERROR "libjpeg found but lacks JCS_EXT_RGBA (need libjpeg-turbo)")
        else()
            message(STATUS "libjpeg found but lacks JCS_EXT_RGBA – disabling JPEG support")
        endif()
    endif()
endif()

# libcurl
q2pro_find_dependency(CURL CURL)
set(HAVE_CURL OFF)
if(CURL_FOUND)
    set(HAVE_CURL ON)
endif()

# SDL2
set(HAVE_SDL2 OFF)
set(_sdl_mode "${Q2PRO_USE_SDL2}")
if(NOT _sdl_mode STREQUAL "OFF")
    if(_sdl_mode STREQUAL "ON")
        find_package(SDL2 REQUIRED)
    else()
        find_package(SDL2 QUIET)
    endif()
    if(SDL2_FOUND)
        set(HAVE_SDL2 ON)
    endif()
endif()

# OpenAL
set(HAVE_OPENAL OFF)
set(_oal_mode "${Q2PRO_USE_OPENAL}")
if(NOT _oal_mode STREQUAL "OFF")
    if(_oal_mode STREQUAL "ON")
        find_package(OpenAL REQUIRED)
    else()
        find_package(OpenAL QUIET)
    endif()
    if(OPENAL_FOUND)
        set(HAVE_OPENAL ON)
    endif()
endif()

# FFmpeg (avcodec, avformat, avutil, swresample, swscale)
set(HAVE_AVCODEC OFF)
set(_av_mode "${Q2PRO_USE_AVCODEC}")
if(NOT _av_mode STREQUAL "OFF")
    if(_av_mode STREQUAL "ON")
        pkg_check_modules(AVCODEC    REQUIRED IMPORTED_TARGET libavcodec>=59.37.100)
        pkg_check_modules(AVFORMAT   REQUIRED IMPORTED_TARGET libavformat>=59.27.100)
        pkg_check_modules(AVUTIL     REQUIRED IMPORTED_TARGET libavutil>=57.28.100)
        pkg_check_modules(SWRESAMPLE REQUIRED IMPORTED_TARGET libswresample>=4.7.100)
        pkg_check_modules(SWSCALE    REQUIRED IMPORTED_TARGET libswscale>=6.7.100)
        set(HAVE_AVCODEC ON)
    else()
        pkg_check_modules(AVCODEC    QUIET IMPORTED_TARGET libavcodec>=59.37.100)
        pkg_check_modules(AVFORMAT   QUIET IMPORTED_TARGET libavformat>=59.27.100)
        pkg_check_modules(AVUTIL     QUIET IMPORTED_TARGET libavutil>=57.28.100)
        pkg_check_modules(SWRESAMPLE QUIET IMPORTED_TARGET libswresample>=4.7.100)
        pkg_check_modules(SWSCALE    QUIET IMPORTED_TARGET libswscale>=6.7.100)
        if(AVCODEC_FOUND AND AVFORMAT_FOUND AND AVUTIL_FOUND AND SWRESAMPLE_FOUND AND SWSCALE_FOUND)
            set(HAVE_AVCODEC ON)
        endif()
    endif()
endif()

# Texture format list
set(Q2PRO_TEXTURE_FORMATS "")
if(HAVE_PNG)
    list(APPEND Q2PRO_TEXTURE_FORMATS "png")
endif()
if(HAVE_JPG)
    list(APPEND Q2PRO_TEXTURE_FORMATS "jpg")
endif()
if(Q2PRO_TGA)
    list(APPEND Q2PRO_TEXTURE_FORMATS "tga")
endif()
list(JOIN Q2PRO_TEXTURE_FORMATS " " Q2PRO_TEXTURE_FORMATS_STR)

# ---------------------------------------------------------------------------
# Platform sources and libraries
# ---------------------------------------------------------------------------
set(PLATFORM_COMMON_SRC "")
set(PLATFORM_CLIENT_SRC "")
set(PLATFORM_SERVER_SRC "")
set(PLATFORM_GAME_SRC "")
set(PLATFORM_COMMON_LIBS "")
set(PLATFORM_CLIENT_LIBS "")
set(PLATFORM_SERVER_LIBS "")
set(PLATFORM_GAME_LIBS "")

set(HAVE_X11 OFF)
set(HAVE_WAYLAND OFF)
set(HAVE_SNDDMA OFF)
set(HAVE_ICMP OFF)

if(Q2PRO_WIN32)
    # ---- Windows platform ----
    list(APPEND PLATFORM_COMMON_SRC windows/system.c windows/hunk.c)
    list(APPEND PLATFORM_CLIENT_SRC windows/client.c windows/wgl.c)
    list(APPEND PLATFORM_COMMON_LIBS ws2_32)
    list(APPEND PLATFORM_CLIENT_LIBS opengl32)

    if(Q2PRO_SOFTWARE_SOUND)
        list(APPEND PLATFORM_CLIENT_SRC windows/wave.c windows/dsound.c)
        list(APPEND PLATFORM_CLIENT_LIBS winmm)
        set(HAVE_SNDDMA ON)
    endif()

    # Windows EGL
    option(Q2PRO_WINDOWS_EGL "Windows EGL video driver" ON)
    if(Q2PRO_WINDOWS_EGL)
        list(APPEND PLATFORM_CLIENT_SRC windows/egl.c)
    endif()

    # Windows service
    option(Q2PRO_WINDOWS_SERVICE "Support for running as Windows service" ON)

    # Windows crash dumps
    option(Q2PRO_WINDOWS_CRASH_DUMPS "Windows crash dumps" OFF)
    if(Q2PRO_WINDOWS_CRASH_DUMPS AND Q2PRO_CPU MATCHES "x86")
        list(APPEND PLATFORM_COMMON_SRC windows/debug.c)
    endif()

else()
    # ---- Unix/Linux/macOS platform ----
    list(APPEND PLATFORM_COMMON_SRC unix/system.c unix/hunk.c)

    find_library(LIBM m)
    if(LIBM)
        list(APPEND PLATFORM_COMMON_LIBS ${LIBM})
        list(APPEND PLATFORM_GAME_LIBS ${LIBM})
    endif()

    find_library(LIBRT rt)
    if(LIBRT)
        list(APPEND PLATFORM_COMMON_LIBS ${LIBRT})
    endif()

    find_library(LIBDL dl)
    if(LIBDL)
        list(APPEND PLATFORM_COMMON_LIBS ${LIBDL})
    endif()

    find_package(Threads REQUIRED)
    list(APPEND PLATFORM_CLIENT_LIBS Threads::Threads)

    # System console (tty)
    if(Q2PRO_SYSTEM_CONSOLE)
        list(APPEND PLATFORM_COMMON_SRC unix/tty.c)
    endif()

    # SDL2 video/sound
    if(HAVE_SDL2)
        list(APPEND PLATFORM_CLIENT_SRC unix/video/sdl.c)
    endif()

    # X11
    set(_x11_mode "${Q2PRO_USE_X11}")
    if(NOT _x11_mode STREQUAL "OFF")
        find_package(X11 QUIET)
        if(X11_FOUND AND X11_Xi_FOUND)
            # Also need GL
            find_package(OpenGL QUIET)
            if(OPENGL_FOUND)
                set(HAVE_X11 ON)
                list(APPEND PLATFORM_CLIENT_SRC
                    unix/video/x11.c
                    unix/video/keytables/evdev.c
                    unix/video/keytables/at.c)
                list(APPEND PLATFORM_CLIENT_LIBS
                    ${X11_LIBRARIES} ${X11_Xi_LIB} OpenGL::GL)
            endif()
        elseif(_x11_mode STREQUAL "ON")
            message(FATAL_ERROR "X11 requested but X11/Xi/GL not found")
        endif()
    endif()

    # Wayland
    set(_wl_mode "${Q2PRO_USE_WAYLAND}")
    if(NOT _wl_mode STREQUAL "OFF")
        pkg_check_modules(WL_CLIENT    QUIET IMPORTED_TARGET wayland-client>=1.18)
        pkg_check_modules(WL_CURSOR    QUIET IMPORTED_TARGET wayland-cursor>=1.18)
        pkg_check_modules(WL_EGL       QUIET IMPORTED_TARGET wayland-egl>=9.0.0)
        pkg_check_modules(WL_PROTOCOLS QUIET IMPORTED_TARGET wayland-protocols>=1.15)
        pkg_check_modules(LIBDECOR     QUIET IMPORTED_TARGET libdecor-0>=0.1.0)
        pkg_check_modules(EGL          QUIET IMPORTED_TARGET egl>=1.5)

        set(_wl_all_found OFF)
        if(WL_CLIENT_FOUND AND WL_CURSOR_FOUND AND WL_EGL_FOUND AND
           WL_PROTOCOLS_FOUND AND LIBDECOR_FOUND AND EGL_FOUND)
            # Check for linux/input-event-codes.h
            check_include_file("linux/input-event-codes.h" HAVE_INPUT_EVENT_CODES)
            if(HAVE_INPUT_EVENT_CODES)
                find_program(WL_SCANNER wayland-scanner)
                if(WL_SCANNER)
                    set(_wl_all_found ON)
                endif()
            endif()
        endif()

        if(_wl_all_found)
            set(HAVE_WAYLAND ON)

            # Get protocols directory
            pkg_get_variable(WL_PROTOCOLS_DIR wayland-protocols pkgdatadir)

            set(_wl_protocols
                "unstable/pointer-constraints/pointer-constraints-unstable-v1.xml"
                "unstable/primary-selection/primary-selection-unstable-v1.xml"
                "unstable/relative-pointer/relative-pointer-unstable-v1.xml"
            )

            set(_wl_gen_sources "")
            set(_wl_gen_headers "")
            foreach(_proto ${_wl_protocols})
                get_filename_component(_basename "${_proto}" NAME_WE)
                set(_xml "${WL_PROTOCOLS_DIR}/${_proto}")
                set(_c_out "${CMAKE_CURRENT_BINARY_DIR}/wayland/${_basename}.c")
                set(_h_out "${CMAKE_CURRENT_BINARY_DIR}/wayland/${_basename}.h")

                add_custom_command(
                    OUTPUT "${_c_out}"
                    COMMAND ${WL_SCANNER} private-code "${_xml}" "${_c_out}"
                    DEPENDS "${_xml}" VERBATIM)
                add_custom_command(
                    OUTPUT "${_h_out}"
                    COMMAND ${WL_SCANNER} client-header "${_xml}" "${_h_out}"
                    DEPENDS "${_xml}" VERBATIM)

                list(APPEND _wl_gen_sources "${_c_out}")
                list(APPEND _wl_gen_headers "${_h_out}")
            endforeach()

            add_library(wl_protocols STATIC ${_wl_gen_sources} ${_wl_gen_headers})
            target_link_libraries(wl_protocols PRIVATE PkgConfig::WL_CLIENT)
            target_include_directories(wl_protocols PUBLIC "${CMAKE_CURRENT_BINARY_DIR}/wayland")

            list(APPEND PLATFORM_CLIENT_SRC unix/video/wayland.c unix/video/keytables/evdev.c)
            list(APPEND PLATFORM_CLIENT_LIBS
                PkgConfig::WL_CLIENT PkgConfig::WL_CURSOR PkgConfig::WL_EGL
                PkgConfig::LIBDECOR PkgConfig::EGL wl_protocols)
        elseif(_wl_mode STREQUAL "ON")
            message(FATAL_ERROR "Wayland requested but dependencies not found")
        endif()
    endif()

    # Software sound on Unix
    if(Q2PRO_SOFTWARE_SOUND)
        if(HAVE_SDL2)
            list(APPEND PLATFORM_CLIENT_SRC unix/sound/sdl.c)
            set(HAVE_SNDDMA ON)
        endif()
    endif()

    # ICMP errors
    if(Q2PRO_ICMP_ERRORS)
        include(CheckCSourceCompiles)
        set(CMAKE_REQUIRED_DEFINITIONS -D_GNU_SOURCE)
        check_c_source_compiles("
            #include <time.h>
            #include <linux/errqueue.h>
            int main(void) { return 0; }
        " HAVE_LINUX_ERRQUEUE)
        unset(CMAKE_REQUIRED_DEFINITIONS)
        if(HAVE_LINUX_ERRQUEUE)
            set(HAVE_ICMP ON)
        endif()
    endif()

    # POSIX function checks
    include(CheckFunctionExists)
    set(CMAKE_REQUIRED_DEFINITIONS -D_GNU_SOURCE)
    check_function_exists(memccpy   HAVE_MEMCCPY)
    check_function_exists(strchrnul HAVE_STRCHRNUL)
    check_function_exists(strnlen   HAVE_STRNLEN)
    unset(CMAKE_REQUIRED_DEFINITIONS)
endif()

# ---------------------------------------------------------------------------
# Conditional sources
# ---------------------------------------------------------------------------
if(HAVE_CURL)
    list(APPEND CLIENT_SRC client/http.c)
endif()

if(HAVE_OPENAL)
    list(APPEND CLIENT_SRC client/sound/al.c client/sound/qal.c)
endif()

if(HAVE_AVCODEC)
    list(APPEND CLIENT_SRC client/cin.c client/sound/ogg.c)
endif()

if(Q2PRO_CLIENT_GTV)
    list(APPEND CLIENT_SRC client/gtv.c)
endif()

if(Q2PRO_CLIENT_UI)
    list(APPEND CLIENT_SRC ${UI_SRC})
endif()

if(HAVE_SNDDMA)
    list(APPEND CLIENT_SRC client/sound/dma.c)
endif()

if(Q2PRO_ANTICHEAT_SERVER)
    list(APPEND SERVER_SRC server/ac.c)
endif()

if(Q2PRO_MVD_SERVER)
    list(APPEND COMMON_SRC server/mvd.c)
endif()

if(Q2PRO_MVD_CLIENT)
    list(APPEND COMMON_SRC
        server/mvd/client.c
        server/mvd/game.c
        server/mvd/parse.c)
endif()

if(Q2PRO_SAVE_GAMES)
    list(APPEND COMMON_SRC server/save.c)
endif()

if(Q2PRO_TESTS)
    list(APPEND COMMON_SRC common/tests.c)
endif()

if(Q2PRO_DEBUG)
    list(APPEND SRC_GL refresh/gl/debug.c)
endif()

# ---------------------------------------------------------------------------
# Default GL profile string
# ---------------------------------------------------------------------------
if(APPLE)
    set(Q2PRO_DEFGLPROFILE "gl3.2")
elseif(Q2PRO_OPENGL_ES1)
    set(Q2PRO_DEFGLPROFILE "es1")
else()
    set(Q2PRO_DEFGLPROFILE "")
endif()

# ---------------------------------------------------------------------------
# Endianness
# ---------------------------------------------------------------------------
include(TestBigEndian)
test_big_endian(Q2PRO_BIG_ENDIAN)
if(Q2PRO_BIG_ENDIAN)
    set(Q2PRO_ENDIAN_DEF "USE_BIG_ENDIAN")
else()
    set(Q2PRO_ENDIAN_DEF "USE_LITTLE_ENDIAN")
endif()

# ---------------------------------------------------------------------------
# Generate config.h
# ---------------------------------------------------------------------------
# Map booleans to 0/1 for cmakedefine01
macro(q2pro_bool_to_01 _var _out)
    if(${_var})
        set(${_out} 1)
    else()
        set(${_out} 0)
    endif()
endmacro()

q2pro_bool_to_01(HAVE_ZLIB        USE_ZLIB)
q2pro_bool_to_01(HAVE_PNG         USE_PNG)
q2pro_bool_to_01(HAVE_JPG         USE_JPG)
q2pro_bool_to_01(HAVE_CURL        USE_CURL)
q2pro_bool_to_01(HAVE_OPENAL      USE_OPENAL)
q2pro_bool_to_01(HAVE_AVCODEC     USE_AVCODEC)
q2pro_bool_to_01(Q2PRO_TGA        USE_TGA)
q2pro_bool_to_01(HAVE_SDL2        USE_SDL)
q2pro_bool_to_01(HAVE_X11         USE_X11)
q2pro_bool_to_01(HAVE_WAYLAND     USE_WAYLAND)
q2pro_bool_to_01(HAVE_SNDDMA      USE_SNDDMA)
q2pro_bool_to_01(Q2PRO_AUTO_REPLY USE_AUTOREPLY)
q2pro_bool_to_01(Q2PRO_DEBUG      USE_DEBUG)
q2pro_bool_to_01(Q2PRO_VARIABLE_FPS USE_FPS)
q2pro_bool_to_01(Q2PRO_OPENGL_ES1 USE_GLES)
q2pro_bool_to_01(HAVE_ICMP        USE_ICMP)
q2pro_bool_to_01(Q2PRO_MD3        USE_MD3)
q2pro_bool_to_01(Q2PRO_MD5        USE_MD5)
q2pro_bool_to_01(Q2PRO_PACKETDUP_HACK USE_PACKETDUP)
q2pro_bool_to_01(Q2PRO_SAVE_GAMES USE_SAVEGAMES)
q2pro_bool_to_01(Q2PRO_SYSTEM_CONSOLE USE_SYSCON)
q2pro_bool_to_01(Q2PRO_CLIENT_UI  USE_UI)
q2pro_bool_to_01(Q2PRO_CLIENT_GTV USE_CLIENT_GTV)
q2pro_bool_to_01(Q2PRO_MVD_SERVER USE_MVD_SERVER)
q2pro_bool_to_01(Q2PRO_MVD_CLIENT USE_MVD_CLIENT)
q2pro_bool_to_01(Q2PRO_TESTS      USE_TESTS)
q2pro_bool_to_01(Q2PRO_GAME_ABI_HACK USE_GAME_ABI_HACK)

# Protocol extensions always on
set(USE_PROTOCOL_EXTENSIONS 1)

# New game API — only define in config.h when explicitly enabled
# (shared.h provides a fallback: USE_NEW_GAME_API = USE_CLIENT || USE_SERVER)
if(Q2PRO_GAME_NEW_API)
    set(USE_NEW_GAME_API 1)
else()
    unset(USE_NEW_GAME_API)
endif()

# Toggles that map to USE_SERVER (defined as string, not 0/1)
if(Q2PRO_ANTICHEAT_SERVER)
    set(USE_AC_SERVER "USE_SERVER")
endif()
if(Q2PRO_WIN32)
    option(Q2PRO_WINDOWS_SERVICE "Support for running as Windows service" ON)
    if(Q2PRO_WINDOWS_SERVICE)
        set(USE_WINSVC "USE_SERVER")
    endif()
endif()

# Windows-specific
if(Q2PRO_WIN32)
    if(Q2PRO_WINDOWS_CRASH_DUMPS AND Q2PRO_CPU MATCHES "x86")
        set(USE_DBGHELP 1)
    else()
        set(USE_DBGHELP 0)
    endif()
    if(Q2PRO_WINDOWS_EGL)
        set(USE_WIN32EGL 1)
    else()
        set(USE_WIN32EGL 0)
    endif()
else()
    set(USE_DBGHELP 0)
    set(USE_WIN32EGL 0)
endif()

configure_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/config.h.in"
    "${CMAKE_CURRENT_BINARY_DIR}/config.h"
    @ONLY
)

include_directories("${CMAKE_CURRENT_BINARY_DIR}")

# ---------------------------------------------------------------------------
# Client executable: q2pro
# ---------------------------------------------------------------------------
set(_client_all_src ${COMMON_SRC} ${CLIENT_SRC} ${REFRESH_SRC} ${SRC_GL}
                    ${PLATFORM_COMMON_SRC} ${PLATFORM_CLIENT_SRC})
list(TRANSFORM _client_all_src PREPEND "${CMAKE_CURRENT_SOURCE_DIR}/")

# Check we have enough to build client
set(_can_build_client ON)
if(NOT Q2PRO_WIN32)
    if(NOT HAVE_SDL2 AND NOT HAVE_X11 AND NOT HAVE_WAYLAND)
        message(WARNING "No video drivers enabled – client will not be built")
        set(_can_build_client OFF)
    endif()
endif()

if(_can_build_client)
    add_executable(q2pro ${_client_all_src})
    target_include_directories(q2pro PRIVATE "${CMAKE_SOURCE_DIR}/inc")
    target_compile_definitions(q2pro PRIVATE USE_CLIENT=1 USE_REF=1)

    if(Q2PRO_WIN32)
        target_compile_definitions(q2pro PRIVATE _WIN32_WINNT=0x0601)
        set_target_properties(q2pro PROPERTIES WIN32_EXECUTABLE TRUE)
    endif()

    if(HAS_WMISSING_PROTOTYPES)
        target_compile_options(q2pro PRIVATE -Wmissing-prototypes)
    endif()

    target_link_libraries(q2pro PRIVATE ${PLATFORM_COMMON_LIBS} ${PLATFORM_CLIENT_LIBS})

    if(HAVE_ZLIB)
        target_link_libraries(q2pro PRIVATE ZLIB::ZLIB)
    endif()
    if(HAVE_PNG)
        target_link_libraries(q2pro PRIVATE PNG::PNG)
    endif()
    if(HAVE_JPG)
        target_link_libraries(q2pro PRIVATE JPEG::JPEG)
    endif()
    if(HAVE_CURL)
        target_link_libraries(q2pro PRIVATE CURL::libcurl)
    endif()
    if(HAVE_SDL2)
        if(TARGET SDL2::SDL2)
            target_link_libraries(q2pro PRIVATE SDL2::SDL2)
        else()
            target_include_directories(q2pro PRIVATE ${SDL2_INCLUDE_DIRS})
            target_link_libraries(q2pro PRIVATE ${SDL2_LIBRARIES})
        endif()
    endif()
    if(HAVE_OPENAL)
        target_link_libraries(q2pro PRIVATE OpenAL::OpenAL)
    endif()
    if(HAVE_AVCODEC)
        target_link_libraries(q2pro PRIVATE
            PkgConfig::AVCODEC PkgConfig::AVFORMAT PkgConfig::AVUTIL
            PkgConfig::SWRESAMPLE PkgConfig::SWSCALE)
    endif()

    set_target_properties(q2pro PROPERTIES
        C_VISIBILITY_PRESET hidden
        RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}")

    if(Q2PRO_SYSTEM_WIDE AND NOT Q2PRO_WIN32)
        install(TARGETS q2pro RUNTIME DESTINATION "${CMAKE_INSTALL_BINDIR}")
    endif()
endif()

# ---------------------------------------------------------------------------
# Dedicated server: q2proded
# ---------------------------------------------------------------------------
set(_server_all_src ${COMMON_SRC} ${SERVER_SRC}
                    ${PLATFORM_COMMON_SRC})
list(TRANSFORM _server_all_src PREPEND "${CMAKE_CURRENT_SOURCE_DIR}/")

add_executable(q2proded ${_server_all_src})
target_include_directories(q2proded PRIVATE "${CMAKE_SOURCE_DIR}/inc")
target_compile_definitions(q2proded PRIVATE USE_SERVER=1)

if(Q2PRO_WIN32)
    target_compile_definitions(q2proded PRIVATE _WIN32_WINNT=0x0601)
endif()

if(HAS_WMISSING_PROTOTYPES)
    target_compile_options(q2proded PRIVATE -Wmissing-prototypes)
endif()

target_link_libraries(q2proded PRIVATE ${PLATFORM_COMMON_LIBS})

if(HAVE_ZLIB)
    target_link_libraries(q2proded PRIVATE ZLIB::ZLIB)
endif()

set_target_properties(q2proded PROPERTIES
    C_VISIBILITY_PRESET hidden
    RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}")

if(Q2PRO_SYSTEM_WIDE AND NOT Q2PRO_WIN32)
    install(TARGETS q2proded RUNTIME DESTINATION "${CMAKE_INSTALL_BINDIR}")
endif()

# ---------------------------------------------------------------------------
# Game shared library: game<cpu>
# ---------------------------------------------------------------------------
set(_game_all_src ${GAME_SRC})
list(TRANSFORM _game_all_src PREPEND "${CMAKE_CURRENT_SOURCE_DIR}/")

add_library(game SHARED ${_game_all_src})
target_include_directories(game PRIVATE "${CMAKE_SOURCE_DIR}/inc")
set_target_properties(game PROPERTIES
    PREFIX ""
    OUTPUT_NAME "game${Q2PRO_CPU}"
    C_VISIBILITY_PRESET hidden
    LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}")

target_link_libraries(game PRIVATE ${PLATFORM_GAME_LIBS})

if(HAVE_ZLIB)
    target_link_libraries(game PRIVATE ZLIB::ZLIB)
endif()

if(CMAKE_C_COMPILER_ID STREQUAL "GNU" AND NOT Q2PRO_WIN32)
    target_link_options(game PRIVATE "-static-libgcc")
endif()

if(Q2PRO_SYSTEM_WIDE AND NOT Q2PRO_WIN32)
    install(TARGETS game
        LIBRARY DESTINATION "${Q2PRO_LIBDIR}/${Q2PRO_BASE_GAME}")
endif()

# ---------------------------------------------------------------------------
# Install data
# ---------------------------------------------------------------------------
if(Q2PRO_SYSTEM_WIDE AND NOT Q2PRO_WIN32)
    if(Q2PRO_CLIENT_UI)
        install(FILES "${CMAKE_CURRENT_SOURCE_DIR}/client/ui/q2pro.menu"
            DESTINATION "${Q2PRO_DATADIR}/${Q2PRO_BASE_GAME}")
    endif()
endif()

# ---------------------------------------------------------------------------
# Feature summary
# ---------------------------------------------------------------------------
message(STATUS "")
message(STATUS "=== Q2PRO Build Configuration ===")
message(STATUS "  Platform:          ${Q2PRO_BUILDSTRING} (${Q2PRO_CPU})")
message(STATUS "  System-wide:       ${Q2PRO_SYSTEM_WIDE}")
if(Q2PRO_SYSTEM_WIDE AND NOT Q2PRO_WIN32)
    message(STATUS "    datadir:         ${Q2PRO_DATADIR}")
    message(STATUS "    libdir:          ${Q2PRO_LIBDIR}")
    message(STATUS "    homedir:         ${Q2PRO_HOMEDIR_CFG}")
endif()
message(STATUS "  zlib:              ${HAVE_ZLIB}")
message(STATUS "  libpng:            ${HAVE_PNG}")
message(STATUS "  libjpeg:           ${HAVE_JPG}")
message(STATUS "  libcurl:           ${HAVE_CURL}")
message(STATUS "  SDL2:              ${HAVE_SDL2}")
message(STATUS "  OpenAL:            ${HAVE_OPENAL}")
message(STATUS "  FFmpeg/avcodec:    ${HAVE_AVCODEC}")
message(STATUS "  X11:               ${HAVE_X11}")
message(STATUS "  Wayland:           ${HAVE_WAYLAND}")
message(STATUS "  Software sound:    ${HAVE_SNDDMA}")
message(STATUS "  Client UI:        ${Q2PRO_CLIENT_UI}")
message(STATUS "  MVD server:        ${Q2PRO_MVD_SERVER}")
message(STATUS "  MVD client:        ${Q2PRO_MVD_CLIENT}")
message(STATUS "  Save games:        ${Q2PRO_SAVE_GAMES}")
message(STATUS "  MD3:               ${Q2PRO_MD3}")
message(STATUS "  MD5:               ${Q2PRO_MD5}")
message(STATUS "  TGA:               ${Q2PRO_TGA}")
message(STATUS "  Debug:             ${Q2PRO_DEBUG}")
if(_can_build_client)
    message(STATUS "  Client:            YES (q2pro)")
else()
    message(STATUS "  Client:            NO (no video driver)")
endif()
message(STATUS "  Server:            YES (q2proded)")
message(STATUS "  Game library:      game${Q2PRO_CPU}")
message(STATUS "")

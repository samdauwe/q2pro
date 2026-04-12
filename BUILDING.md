Prerequisities
--------------

Q2PRO can be built on Linux, BSD and similar platfroms using a recent version
of GCC or Clang.

Q2PRO client requires either SDL2 or OpenAL for sound output. For video output,
native X11 and Wayland backends are available, as well as generic SDL2 backend.

Note that SDL2 is optional if using native X11 and Wayland backends and OpenAL,
which is preferred configuration.

Both client and dedicated server require zlib support for full compatibility at
network protocol level. The rest of dependencies are optional.

For JPEG support libjpeg-turbo is required, plain libjpeg will not work. Most
Linux distributions already provide libjpeg-turbo in place of libjpeg.

For playing back music and cinematics FFmpeg libraries are required.

OpenAL sound backend requires OpenAL Soft development headers for compilation.
At runtime, OpenAL library from any vendor can be used (but OpenAL Soft is
strongly recommended).

To install the *full* set of dependencies for building Q2PRO on Debian or
Ubuntu use the following command:

    apt-get install cmake ninja-build gcc libc6-dev libsdl2-dev libopenal-dev \
                    libpng-dev libjpeg-dev zlib1g-dev mesa-common-dev \
                    libcurl4-gnutls-dev libx11-dev libxi-dev \
                    libwayland-dev wayland-protocols libdecor-0-dev \
                    libavcodec-dev libavformat-dev libavutil-dev \
                    libswresample-dev libswscale-dev

If you intend to build just dedicated server, smaller set of dependencies can
be installed:

    apt-get install cmake ninja-build gcc libc6-dev zlib1g-dev


Building
--------

Q2PRO uses CMake for its build process.

Configure the build (Ninja is recommended but not required):

    cmake -B build -G Ninja

To review and change options, use `ccmake` or pass `-D` flags on the command
line. Project-specific options are prefixed with `Q2PRO_`. For example, to
change the install prefix:

    cmake -B build -G Ninja -DCMAKE_INSTALL_PREFIX=/usr

Optional dependencies can be toggled with `AUTO` (default), `ON`, or `OFF`:

    cmake -B build -G Ninja -DQ2PRO_USE_AVCODEC=ON -DQ2PRO_USE_SDL2=OFF

Build:

    cmake --build build

To enable verbose output during the build:

    cmake --build build --verbose


Installation
------------

You need to have either full version of Quake 2 unpacked somewhere, or a demo.
Both should be patched to 3.20 point release.

Run `sudo cmake --install build` to install Q2PRO system-wide into the
configured prefix (`/usr/local` by default).

Copy `baseq2/pak*.pak` files and `baseq2/players` directory from unpacked
Quake 2 data into `/usr/local/share/q2pro/baseq2` to complete the
installation.

Alternatively, configure with `-DQ2PRO_SYSTEM_WIDE=OFF` to build a 'portable'
version that expects to be launched from the root of Quake 2 data tree (this
is default when building for Windows).

On Windows, Q2PRO automatically sets current directory to the directory Q2PRO
executable is in. On other platforms current directory must be set before
launching Q2PRO executable if portable version is built.


Music support
-------------

Q2PRO supports playback of background music ripped off original CD in Ogg
Vorbis format. Music files should be placed in `music` subdirectory of the game
directory in format `music/trackNN.ogg`, where `NN` corresponds to CD track
number. `NN` should be typically in range 02-11 (track 01 is data track on
original CD and should never be used). GOG naming scheme which has tracks 02-21
(with extra tracks for `rogue` and `xatrix` addons) is also supported.

Depending on FFmpeg configuration, music in several other formats can be
transparently supported: FLAC, Opus, MP3 and WAV.

MinGW-w64
---------

MinGW-w64 cross-compiler is available in recent versions of all major Linux
distributions.

To install MinGW-w64 on Debian or Ubuntu, use the following command:

    apt-get install cmake ninja-build mingw-w64

It is recommended to also install nasm, which is needed to build libjpeg-turbo
with SIMD support:

    apt-get install nasm

Create a CMake toolchain file (e.g. `x86_64-w64-mingw32.cmake`):

    set(CMAKE_SYSTEM_NAME Windows)
    set(CMAKE_SYSTEM_PROCESSOR x86_64)
    set(CMAKE_C_COMPILER x86_64-w64-mingw32-gcc)
    set(CMAKE_RC_COMPILER x86_64-w64-mingw32-windres)
    set(CMAKE_FIND_ROOT_PATH_MODE_PROGRAM NEVER)
    set(CMAKE_FIND_ROOT_PATH_MODE_LIBRARY ONLY)
    set(CMAKE_FIND_ROOT_PATH_MODE_INCLUDE ONLY)

Configure and build:

    cmake -B build -G Ninja \
        -DCMAKE_TOOLCHAIN_FILE=x86_64-w64-mingw32.cmake \
        -DCMAKE_BUILD_TYPE=Release
    cmake --build build


Visual Studio
-------------

Q2PRO can be built on Windows using Visual Studio 2022.

Open the Q2PRO source directory using `File > Open > Folder` in Visual Studio.
Visual Studio has built-in CMake support and will automatically detect the
`CMakeLists.txt` file.

Alternatively, from the `x64 Native Tools Command Prompt`:

    cmake -B build -G Ninja -DCMAKE_BUILD_TYPE=Release
    cmake --build build

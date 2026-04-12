Q2PRO
=====

Q2PRO is an enhanced, open-source Quake 2 client and server focused on
multiplayer. It runs on Windows, Linux, macOS, and FreeBSD.

Q2PRO builds upon the original Quake 2 engine with a modernized codebase,
improved networking, and extended renderer capabilities while maintaining full
compatibility with the original game data. It also has work-in-progress support
for the Nightdive remaster assets.

## Features

* Unified OpenGL renderer with support for a wide range of OpenGL versions
* OpenGL ES 1.x renderer for embedded/mobile platforms
* Enhanced console with persistent command history and autocompletion
* Rendering / physics / packet rate separation
* GPU sync for reduced input lag with vsync on
* ZIP packfiles (.pkz)
* JPEG/PNG textures and screenshots
* MD3 and MD5 (re-release) models
* Ogg Vorbis music and Ogg Theora cinematics via FFmpeg
* Compatibility with re-release assets
* Fast and secure HTTP downloads via libcurl
* Multichannel sound using OpenAL
* Stereo WAV files support
* Seeking in demos, recording from demos, server side multiview demos
* Live game broadcasting capabilities (MVD/GTV)
* Network protocol extensions for larger maps
* Native X11 and Wayland video backends (in addition to SDL2)
* Eliminates frame overflows (even for legacy clients)
* Won't crash if game data is corrupted

## Supported Mods

Q2PRO's built-in game module is compatible with the base game. Third-party game
libraries can be used for mission packs and mods, including:

* **The Reckoning** (Xatrix)
* **Ground Zero** (Rogue)
* **Zaero**
* **Capture the Flag**
* **Nightdive Remaster** (work in progress)

## Downloads

Q2PRO doesn't have versioned releases. It is always recommended to use the
latest nightly build from the top of the
[Releases](https://github.com/skullernet/q2pro/releases) page.

Linux binaries are not provided. Users are advised to build from source. See
[BUILDING.md](BUILDING.md) for instructions.

## Documentation

* [Building from source](BUILDING.md)
* [Client manual](doc/client.asciidoc)
* [Server manual](doc/server.asciidoc)

## License

Q2PRO is licensed under the terms of the **GNU General Public License v2.0**.
See [LICENSE](LICENSE) for the full license text.

The Quake II game data files remain copyrighted and licensed under the original
id Software terms. You must own a copy of Quake 2 (retail or demo, patched to
3.20) to play.

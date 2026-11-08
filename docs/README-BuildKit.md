# NX2navi build kit

This kit builds *The Legend of Zelda: Ocarina of Time* (Nintendo Switch 2, game version 1.1.1) as a native Windows or
Linux program **on your PC, from your own copy of the game**. The kit contains no game code or data. Everything the
build writes (`ocarina.exe` or `ocarina`, the RomFS image, the shader cache, your saves) comes from your game: keep it
to yourself.

The NX2navi launcher (`NX2navi-Setup.exe`, [launcher.md](launcher.md)) does everything below for you, the
downloads included, and keeps the game up to date. This page is the same build by hand.

## What you need

- 64-bit Windows 10 or 11, or x86-64 Linux 6.7 or later, 16 GB of RAM, a Vulkan 1.3 graphics card with mesh shaders and
  a current driver (tested on NVIDIA and AMD).
- Your own dump of the game as a `.nup` file (or its `exefs` and `romfs` folders), with the 1.1.1 update: program
  `0400486024DEC000`, main build `e0f3ccfa`. Other versions are refused.
- About 30 GB free on the disk you build to, besides the dump. The RomFS image alone is 26 GB.

## One-time setup (about 15 minutes)

1. **Python 3.12 or later** from [python.org](https://www.python.org/downloads/windows/). Tick "Add python.exe to PATH"
   in the installer. Then, in a Command Prompt:

   ```bat
   py -m pip install numpy lz4 zstandard zbic
   ```

   `zbic` unpacks the game's compressed program files. It is a separate GPL-2.0 package you install yourself; the kit
   only calls it.

2. **MSYS2** from [msys2.org](https://www.msys2.org/), installed to the default `C:\msys64`. Open **MSYS2 UCRT64** from
   the Start menu and paste:

   ```sh
   pacman -S --needed mingw-w64-ucrt-x86_64-clang mingw-w64-ucrt-x86_64-gcc mingw-w64-ucrt-x86_64-sdl3 mingw-w64-ucrt-x86_64-opus mingw-w64-ucrt-x86_64-libvpx mingw-w64-ucrt-x86_64-dav1d mingw-w64-ucrt-x86_64-zstd mingw-w64-ucrt-x86_64-vulkan-headers mingw-w64-ucrt-x86_64-vulkan-loader mingw-w64-ucrt-x86_64-vulkan-memory-allocator
   ```

   Answer `Y` when it asks. If MSYS2 is somewhere other than `C:\msys64`, add `--msys <folder>\ucrt64` to the build
   command.

3. **Linux:** the system's clang, lld and Ninja take the place of MSYS2, and the libraries come from the distribution.
   On Arch, with Python 3.12 or later and the .NET 10 SDK (the shader translator is built with it):

   ```sh
   sudo pacman -S --needed clang lld ninja sdl3 opus libvpx dav1d zstd vulkan-headers dotnet-sdk
   python3 -m venv .venv
   .venv/bin/pip install numpy lz4 zstandard zbic
   .venv/bin/python tools/build_xlate.py
   ```

   `vk_mem_alloc.h` comes from Vulkan Memory Allocator when the system has it; otherwise the build fetches the pinned
   header into `toolchain/vma` on its first run. The launcher and the setup are Windows-only; on Linux the build is the
   command below and the game is `ocarina` (no `.exe`).

## Build (about 10 to 15 minutes, once)

Unpack this kit anywhere, open a Command Prompt in its folder and run:

```bat
py -I tools\make_game.py --nup "C:\path\to\your game.nup" --out "C:\Games\Ocarina"
```

On Linux, from the kit's folder:

```sh
.venv/bin/python tools/make_game.py --nup "/path/to/your game.nup" --out "$HOME/Games/Ocarina"
```

With extracted folders instead, use `--exefs "C:\path\exefs" --romfs "C:\path\romfs"` in place of `--nup` (POSIX paths
on Linux). To only check that your dump is the right version, add `--check` (it builds nothing).

The build checks the version, unpacks the program files, packs the 26 GB RomFS image (every file is checked against
the dump's own checksums), turns the game's code into C and compiles it (about 6 minutes on a 16-thread Ryzen 7; longer on fewer cores),
then prepares every one of the game's shaders for your graphics card (about 5 more minutes), so the game does not stop
to compile shaders while you play.
You can use your PC meanwhile. If it stops,
run the same command again: finished steps are skipped.

When it says `finished`, start `C:\Games\Ocarina\ocarina.exe` (`$HOME/Games/Ocarina/ocarina` on Linux). You can delete
`C:\Games\Ocarina\build-cache` (about
5 GB) afterwards.

## Playing

- **F1** opens the settings menu. Settings are saved in `nx2navi.ini` beside the game; saves go to `profile\`
  (`profile/` on Linux).
- Every shader was prepared during the build. After an update of your graphics driver, the next start shows
  "Preparing shaders" once while the driver rebuilds them.
- **Screen** in the menu has Upscaling (DLSS on NVIDIA RTX cards, FSR 3.1 on any card), Frame generation (FSR 3.1; it
  needs DLSS or FSR 3.1 upscaling on), HDR (Native is the game's own HDR; turn HDR on in Windows first) and the
  resolution. Those marked "after restart" take effect when the game starts again.
- DLSS needs `nvngx_dlss.dll` from NVIDIA beside `ocarina.exe` (the kit does not include it); FSR 3.1's
  `amd_fidelityfx_vk.dll` is copied there by the build. On Linux neither is available: the game uses FSR 1 or bilinear
  scaling.
- Logs are in `logs\` (`logs/` on Linux). When reporting a problem, say where you were in the game and which graphics
  card you have, and attach `logs\nx2navi.log`. Never send game files.

## If something goes wrong

| Message | What to do |
|---|---|
| `not game version 1.1.1` | Install the 1.1.1 update and dump the game again |
| `the Python package ... is missing` | Run the `pip install` line above |
| `...\clang.exe is missing` | Run the `pacman` line above in MSYS2 UCRT64 |
| `needs ... GB free` | Free space on the `--out` disk, or build to another one |
| The game opens and closes | Update your graphics driver, then send `logs\nx2navi.log` |

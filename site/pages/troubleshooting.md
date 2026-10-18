---
title: Troubleshooting
section: Playing
order: 4
description: Fixes for rejected disc images, failed builds, startup problems, controllers and crashes.
---

# Troubleshooting

## The ISO is rejected

Only the USA retail base disc (`MS-2505`) is supported. A bad dump, another region, a title update or a modified image won't match. The launcher doesn't accept near matches and can't download or repair an image. On Linux and macOS, setup from a terminal also accepts the Games on Demand release of the same build, whose re-signed executables are listed in `config/supported-dumps.json`.

## A download fails

Check the connection and run the launcher again. Network failures are retried, and every pinned download is verified by SHA-256 before use, so partial files are never trusted. If curl fails, the launcher retries without a strict certificate revocation check and then with Windows' own downloader, which follows the system proxy. If a file still won't download but your browser can fetch it (a proxy or antivirus web filtering is the usual cause), download it in the browser and save it under the name and folder the error gives; setup checks it and continues.

## The build fails

Restart Windows after a Build Tools install, free at least 30 GB, and close any running game before trying again. When a step fails, the launcher shows which one, its exit code and the first real compiler, CMake or file-copy error, with a hint for common causes: a full disk, low memory, antivirus, a file in use. The same report is saved in `.local/logs/setup-error.json`. Attach it to a bug report; the final "build failed" line alone can't identify the cause.

When the Visual Studio Installer stops, the error gives its exit code and, for the common ones, the reason: the installer or Visual Studio is already open, or another installation is running. The installer's own logs, the `dd_*.log` files in `%TEMP%`, explain the other codes. The launcher copies the ones from the failed attempt to `.local/logs/build-tools-installer`; attach them with `setup-error.json`. If the installer keeps failing, open **Visual Studio Installer** from the Start menu, repair or remove an incomplete Build Tools install, and restart Windows. Or install the Build Tools yourself with **Desktop development with C++**; the launcher uses an existing install.

## A portable install can't start or build

- **"Portable folder is not writable"**: move the folder somewhere you own, such as `D:\Games\PinyonShift`, or delete `portable.txt`.
- **"Portable folder path is too long"**: the build creates files about 185 characters below `data`. Keep the `data` folder's path at 70 characters or fewer.

## The game doesn't start

Update the GPU driver and check that the GPU supports Vulkan 1.3. To reset the runtime settings, delete `.local/preview/config/pinyon_shift.toml`. Security software may quarantine the newly compiled, unsigned executable. Restore it only if it was built by your own install.

## A white window after the first screen

Up to 0.5.1, a `pinyon_shift.toml` without a `gpu_backend` line started the game on Direct3D 12, which has no prepared shaders on a player's install: the first warning screen showed, then the window stayed white while the game and its audio ran ([#627](https://github.com/arcanite24/pinyon-shift/issues/627)). The game now uses Vulkan when the line is missing. On an older build, add `gpu_backend = "vulkan"` to `.local/preview/config/pinyon_shift.toml`, or delete the file to reset the settings.

## "Could not build the mods' archive members"

Before each start the launcher rebuilds the enabled mods' archives in `.local/preview/mods/zz-archive-patches`. When the mods haven't changed it keeps the previous copy; otherwise it replaces it, and a file still open elsewhere stops it with "is in use by another program". Wait for a game that is still closing, or for antivirus to finish scanning the folder, and start again. If it keeps happening, add the install folder to the antivirus exclusions.
## The game starts but shows nothing (macOS)

The first start after an install or an update builds every pipeline the game saved from earlier sessions before it shows a frame, and macOS compiles each one from scratch the first time (the next start finds them in the system's Metal shader cache). On an M1 that is about 20 seconds for 600 pipelines; a line at the bottom of the window counts them, "LOADING SHADERS 86 / 611", and the same progress goes to `.local/preview/logs/runtime.log` as "Shader status". Wait for it; the second start of the same build takes under a second. The line also appears during play as "COMPILING SHADERS n LEFT" when a scene needs shaders the cache has not seen.

If the line never appears and the log ends after "Initializing shader storage", the machine is short of memory (swap in use after a long session of other work, or a second copy of the game still running). Quit it, make sure no other `pinyon_shift` process is running (Activity Monitor), and start again. Deleting `.local/preview/cache/shaders` only costs one slower first drive while shaders are compiled again.

## Fullscreen is black, windowed is blue (macOS, builds before 2026-10-18)

Going fullscreen with the green button or <kbd>Ctrl</kbd>+<kbd>Cmd</kbd>+<kbd>F</kbd> showed black, and coming back showed a blue window, while the sound and the controller kept working. macOS animates the window into its fullscreen Space, and the game rebuilt its swapchain while the animation was still running, against a layer state the end of the transition replaces; the frames kept presenting and nothing reached the screen until something else rebuilt the connection (opening a menu that changed a display setting did). Fixed: the window now rebuilds the connection when SDL reports the transition finished. Apple's performance HUD grows when the window goes fullscreen on a Retina panel, since it scales with the layer's pixel size, and keeps that size when the window comes back. Apple exposes no setting for the size (only on, off and logging); pick a smaller size from the HUD's own menu-bar item.

## The game runs on the wrong GPU

On a PC with two GPUs, such as a laptop, the game asks for the high-performance one. To pick another with Vulkan, find the "Available Vulkan physical devices" list near the top of `.local/preview/logs/runtime.log` and add its number to `.local/preview/config/pinyon_shift.toml`, for example `vulkan_device = 1`. Direct3D 12 takes `d3d12_adapter`, numbered in Windows' adapter order. `-1` goes back to the automatic choice. Windows' **Settings → System → Display → Graphics** can also assign `pinyon_shift.exe` (not the launcher) to a GPU.

## Stutter or something missing the first time

Vulkan prepares shaders during setup, and stores any new ones it meets during play, so a new effect can stutter once. If stutter repeats on every pass of the same route, or something stays missing after a relaunch, report it with the latest runtime log and performance CSV from `.local/preview/logs` (the launcher's **Logs** link opens it once the game has run).

## The picture tears

With **F6 > DISPLAY > VSYNC** on, the game presents without tearing on both Vulkan and Direct3D 12, and the change applies at once. With it off, Vulkan presents as soon as a frame is ready and may tear. On a G-SYNC or FreeSync display keep VSYNC on as well: with it off and **FRAME RATE LIMIT** at the display's refresh rate (120 at 120 Hz), frames that come a little early are above the variable-refresh range and tear. Overlays that hook the game's presentation, such as RivaTuner or OBS's game capture, can add tearing or hitches of their own: close them to compare.

## A resolution scale error at startup

The renderer supports 1x, 2x, 3x and 4x, the same horizontally and vertically. Pick a supported scale in the launcher, or delete `pinyon_shift.toml` to reset.

## A controller isn't recognized

Connect it before starting the game. Close Steam or other remapping software, reconnect and relaunch. When reporting an unsupported controller, include its exact name, USB vendor and product IDs, and a screenshot from a gamepad tester.

## A DualSense's triggers or lightbar do nothing

Steam Input is probably holding the controller: when Steam starts the game with Steam Input on, the game only sees Steam's virtual Xbox controller, and **F6 > CONTROLS > DUALSENSE** says **STEAM INPUT HAS THE CONTROLLER**. In Steam, open the game's **Properties > Controller**, choose **Disable Steam Input**, and restart the game. Started outside Steam, the game drives the controller itself. See [DualSense](playing.md#dualsense).

## A menu item doesn't respond to the mouse

The Xbox menus don't use pointer targeting. Use A, <kbd>Space</kbd> or a left click to activate the selected row, and <kbd>Enter</kbd> for Start.

## A save copied from Xenia crashes the game

Xenia saves are signed with the Xenia profile's XUID. Import them with `tools/import-xenia-save.ps1` instead of copying; see [Saves](playing.md#saves).

## A Xenia save says the title update is too low

The save was made with title update v4, which the base build can't load. Import your own title update v4 in the launcher's DLC panel (**Import update**), turn on **Use v4**, and play again. The launcher stops before starting the base build with such a save and says the same. See [Title update v4](dlc.md#title-update-v4).

## The game crashes

Leave the launcher open while playing. After a crash it prepares a sanitized ZIP under `.local/preview/reports`, selects it in Explorer and opens a prefilled GitHub issue. Attach the ZIP and describe what you did last. Reports never include the game, your saves or local paths.

When the crash was inside the graphics card's Vulkan driver, as most reported crashes on Intel graphics are, the launcher says so: install the newest driver from the GPU maker's site, and on integrated graphics choose **Low-spec 60** in Settings.

When the game's log shows the graphics card running out of memory, the launcher says that instead. Higher resolution scales need much more graphics memory: a report at 3x ran out within 30 seconds ([#558](https://github.com/arcanite24/pinyon-shift/issues/558)). Choose 1x or 2x, or **Low-spec 60**, in Settings.

Memory dumps in `.local/preview/crashes` can hold fragments of process memory, so they never go into the public report. Keep them local.

## Start over

Close the launcher and delete `.local` and `out` from the source folder under `%LOCALAPPDATA%\PinyonShift` (or under `data\source` for a portable install). Your ISO is outside these folders and is never deleted.

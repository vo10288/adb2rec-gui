# adb2rec-gui

**Android screen acquisition, video/audio recording and evidence export — a Python/Tkinter GUI for Linux and Tsurugi Linux.**

**Author:** Antonio "Visi@n" Broi  
**Contact:** antonio@tsurugi-linux.org  
**Project ecosystem:** [Tsurugi Linux](https://tsurugi-linux.org)  
**License:** MIT  
**Status:** v0.6 Beta Integrated (updated source)

> **Important:** The screen-recording feature requires **up-to-date, mutually compatible `scrcpy`, `scrcpy-server`, and `adb`** components. Installing only a newer `scrcpy` executable while keeping an incompatible or outdated server is not sufficient. `ffmpeg` is also required for the post-processing features.

## Overview

`adb2rec-gui` is a Linux desktop tool for capturing Android device screenshots and recording the device screen. It uses Android Debug Bridge (`adb`) for connection and still-image capture, and `scrcpy` for **video recording with internal Android audio**. It can archive collected material, calculate cryptographic hashes of the resulting archives, and create a per-file SHA-256 integrity manifest.

The GUI is designed for authorized testing, documentation, mobile forensic workflows, training and research. It is **not** a complete forensic acquisition suite and does not independently establish a forensic chain of custody.

## Features

| Feature | Description |
| --- | --- |
| USB connection | Detect an ADB-authorized Android device and switch ADB to TCP/IP port `7777` when requested. |
| Wi-Fi connection | Connect to an Android device via `adb connect <device-ip>:7777`, after USB-side setup. |
| Continuous screenshots | Repeatedly capture PNG images using `adb exec-out screencap -p`. |
| Screenshot UP / DOWN | Capture sequential PNGs while sending automatic swipe gestures to scroll content. |
| Screen + internal audio recording | Record MP4 video with H.264 video and AAC internal audio using an updated `scrcpy`. |
| Timed recording segments | Automatically start successive recording files, with **up to 180 seconds per segment**. |
| Video repair | Reprocess recorded or previously captured MP4 files using FFmpeg. |
| OCR-friendly video | Generate lower-frame-rate, 720p video variants for downstream OCR workflows when repairing existing videos. |
| Archive + hash | Create `.tar.gz` archives, calculate **MD5/SHA-256 hashes of archives**, and generate **SHA-256 digests for individual acquisition files** in `SHA256SUMS.txt`. |
| Session directories | Store acquisitions in timestamped directories and organize exported material using the Android ID when available. |

### What's new in this release

- **Internal audio support** in screen recordings (`--audio-source=output`, `--audio-codec=aac`, `--require-audio`).
- **Automatic splitting into 3-minute recording segments**, rather than a single long recording.
- **Automatic MP4 post-processing** with FFmpeg and retention of relevant scrcpy logs.
- **Original MP4 preservation** (`original_N.mp4`) before FFmpeg transcoding; processed videos are written separately (`N.mp4`).
- **SHA-256 per-file manifest** created before archive generation; allows subsequent file-by-file integrity verification.
- Existing **SCREENSHOT**, **SCREENSHOT UP**, **SCREENSHOT DOWN**, USB/Wi-Fi connection, archive and hashing functions remain available.

## Requirements

- **Linux** with a graphical desktop session (primarily intended for Tsurugi Linux / Ubuntu-based distributions).
- **Python 3** with `tkinter` (the program uses Python standard-library modules; no additional `pip` packages are required by the supplied source).
- **Android SDK Platform-Tools / `adb` — updated** and available on `PATH`.
- **`scrcpy` — updated**, supporting at least `--require-audio` and `--time-limit`.
- **`scrcpy-server` — updated and compatible with the installed `scrcpy` executable** (prefer an official matching release/package).
- **FFmpeg** (`ffmpeg`) for MP4 repair, remuxing/transcoding and OCR-optimized output.
- Linux utilities: `tar`, `md5sum`, `sha256sum`, `cp`, `mv`.
- A correctly configured system locale (the updated script uses the system default locale).
- Optional: `jp2a` and the Tsurugi icon used by the startup banner.

**Device preparation:** enable Developer options, USB debugging, accept the host authorization prompt and keep the device unlocked where required. Root access is not required for the supported ADB/scrcpy workflows.

**Audio compatibility:** audio forwarding/recording via scrcpy generally requires **Android 11+**. Android 11 may need the screen unlocked at the start of the session. For Android 10 and below, this program's `--require-audio` setting prevents video-only fallback. Android audio restrictions and application policies may prevent some sounds from being captured.

## Install (Ubuntu / Tsurugi Linux)

Install the system packages available for your distribution:

```bash
sudo apt update
sudo apt install python3 python3-tk ffmpeg adb locales tar coreutils
```

> A distribution's `scrcpy` package might be older than the version needed here. Download a **matching, current `scrcpy` + `scrcpy-server` distribution from the official project**, or build them following its instructions: <https://github.com/Genymobile/scrcpy>. Do not mix server and client versions from unrelated releases. Platform-Tools (`adb`) should likewise be updated if the packaged version is outdated: <https://developer.android.com/tools/releases/platform-tools>.

Check the main tools:

```bash
adb version
scrcpy --version
scrcpy --help | grep -E -- '--require-audio|--time-limit'
ffmpeg -version
adb devices -l
```

If the desktop locale is misconfigured, generate a suitable UTF-8 locale using your distribution’s locale tools.

### Using a portable scrcpy build

The program chooses `scrcpy` in this order:

1. Executable set using the `SCRCPY_BINARY` environment variable.
2. A recent executable matching `~/scrcpy-linux-x86_64-v*/scrcpy`.
3. `scrcpy` found on `PATH`.

For example:

```bash
export SCRCPY_BINARY="$HOME/scrcpy-linux-x86_64-vX.Y/scrcpy"
python3 adb2rec-gui.py
```

Replace `vX.Y` with the **actual installed folder** and `adb2rec-gui.py` with the filename in your repository. Keep any associated `scrcpy-server` in the location expected by that scrcpy distribution; consult its official packaging instructions. Avoid copying only the executable into `/usr/bin` if its matching server and runtime dependencies are stored elsewhere.

## Run

```bash
python3 adb2rec-gui.py
```

Alternatively, use the updated portable Python shebang:

```bash
chmod +x adb2rec-gui.py
./adb2rec-gui.py
```

**Note:** The updated script uses `#!/usr/bin/env python3`, and the GUI/banner identify version `v0.6 Beta`.

## Basic workflow

1. Connect an Android device over USB and verify that `adb devices -l` reports it as `device`.
2. Launch the GUI. If Wi-Fi operation is needed, choose **ADB CONNECT CABLE** to enable ADB TCP/IP on port `7777`, then **ADB CONNECT WIFI** and enter the device IP address. On some Android versions, this legacy TCP/IP workflow may require additional device-specific setup.
3. Use **SCREENSHOT** for continuous PNG capture, or **SCREENSHOT UP / DOWN** for repeated screenshots with automatic swipes. Stop with **STOP SCREENSHOT**.
4. Use **SCREEN RECORD** for video **and internal audio**. The tool invokes scrcpy, records up to **180 seconds per MP4 segment**, then begins the next segment. Stop with **STOP SCREEN RECORD**. Capturing audio is mandatory in this implementation.
5. Optionally select **REPAIR EXISTING VIDEOS** to repair stored MP4 files and produce OCR-oriented variants.
6. Select **COMPRESS FILES AND CALCULATE HASH** to produce archives and calculate MD5/SHA-256 digests.

### Recording settings in the source

```text
Video codec:       H.264
Video bitrate:     4 Mbit/s
Audio source:      Android internal output
Audio codec:       AAC
Audio requirement: Mandatory (--require-audio)
Segment limit:     180 seconds
Control:           Disabled for the scrcpy recording session (--no-control)
Container:         MP4
Post-processing:   FFmpeg
```

A separate scrcpy window may appear, depending on the build and environment; the supplied recording command does not request headless operation.

## Output layout

The default working directory is **hardcoded in the supplied source**:

```text
~/02.computer_vision/04.video2ocr/
├── screenshot/
│   └── YYYYMMDDHHMMSS/
│       ├── 1.png
│       ├── 2.png
│       └── ...
├── screenrecord/
│   └── YYYYMMDDHHMMSS/
│       ├── original_1.mp4  # untouched scrcpy recording
│       ├── 1.mp4           # FFmpeg derivative
│       ├── original_2.mp4
│       ├── 2.mp4
│       ├── scrcpy_1.log
│       └── ...
├── repaired_videos/
├── compressfiles/
└── hashfiles/
    └── SHA256SUMS.txt
```

The screenshot routine also updates `temp.png` and `temp1.png` in the base directory. During video creation, temporary files named `temp_N.mp4` are used. Successful captures are preserved as `original_N.mp4` before FFmpeg produces the derived `N.mp4`. Logs and partial files may remain after failures.

On completion of **COMPRESS FILES AND CALCULATE HASH**, the program attempts to move archives, hash reports, acquisition directories and temporary PNGs into a subdirectory named for the device's **Android ID** (if retrievable). Actual output may vary according to permissions and existing directories.

**Integrity scope:** alongside MD5/SHA-256 hashes of TAR.GZ archives, the updated script computes a `SHA256SUMS.txt` manifest covering files in `screenshot/`, `screenrecord/`, and `repaired_videos/` **at the time the archive action is selected**. Thus both original `original_N.mp4` and transcoded `N.mp4` get distinct SHA-256 hashes. The originals are preserved, but no acquisition procedure is by itself proof of a complete forensic chain of custody. Record device state, times, tool versions, operator actions and handling history separately.

To verify, first change to the acquisition directory that contains `SHA256SUMS.txt` and the named folders, then run:

```bash
sha256sum -c SHA256SUMS.txt
```

Do not edit captured files after hashing; if additional recordings are made, regenerate the manifest and clearly document the additional acquisition event.

## Troubleshooting

- **`scrcpy is too old`**: verify `SCRCPY_BINARY` and check that your chosen executable implements `--require-audio` and `--time-limit`.
- **Server/client mismatch**: install the scrcpy client and its *matching* `scrcpy-server` from the same official release. Verify that no old package or environment setting shadows them.
- **No audio / recording aborts**: check Android version, unlock state, active audio output, permissions and restrictions. Because `--require-audio` is set, a video-only recording is *not* accepted as a substitute.
- **Device not detected**: use `adb devices -l`, confirm USB debugging and the host's authorization, and check permissions/cable quality.
- **Video repair fails**: check `ffmpeg -version`, inspect `scrcpy_N.log` and make sure the output volume has sufficient free space.
- **Locale error**: verify that the host has a valid UTF-8 system locale.
- **Wrong Python interpreter**: install Python 3 and run with `python3 adb2rec-gui.py`.

## Forensic and legal considerations

Only capture devices and information for which you have lawful authorization. Screenshots, touch/swipe actions, USB debugging and the use of scrcpy may modify device state. This utility is intended as an **acquisition and documentation aid**, not as a guarantee of physical/logical extraction, evidentiary completeness, immutable originals or court admissibility. Apply the procedures required by your organization and jurisdiction.

## Third-party software and attribution

`adb2rec-gui` invokes third-party software but does not grant rights to redistribute it under this project's MIT license. Each dependency retains its own license. **Official download and documentation links:**

- **scrcpy + bundled matching scrcpy-server:** [official repository](https://github.com/Genymobile/scrcpy), [releases / binaries](https://github.com/Genymobile/scrcpy/releases), [official Linux installation guide](https://github.com/Genymobile/scrcpy/blob/master/doc/linux.md). Download the complete Linux release archive; do not mix incompatible client/server builds.
- **ADB (Android SDK Platform-Tools):** [official Android developer downloads](https://developer.android.com/tools/releases/platform-tools), with Linux, macOS and Windows packages.
- **FFmpeg:** [official download page](https://ffmpeg.org/download.html), [official source repository](https://git.ffmpeg.org/ffmpeg.git). On Ubuntu/Tsurugi, prefer the distro package via `sudo apt install ffmpeg` when compatible.
- **Python 3:** [official Python downloads](https://www.python.org/downloads/); **Tkinter:** [Python documentation](https://docs.python.org/3/library/tkinter.html). On Ubuntu/Tsurugi: `sudo apt install python3 python3-tk`.
- **GNU coreutils (`md5sum`, `sha256sum`, `mv`, `cp`):** [GNU Coreutils](https://www.gnu.org/software/coreutils/); **GNU tar:** [GNU tar](https://www.gnu.org/software/tar/).
- **Optional jp2a ASCII banner:** [jp2a project](https://github.com/cslarsen/jp2a).

## Author and license

Copyright (c) 2026 **Antonio "Visi@n" Broi**.

Released under the [MIT License](LICENSE). See `LICENSE` for full terms.

Project ecosystem: [https://tsurugi-linux.org](https://tsurugi-linux.org).

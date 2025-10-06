# Doudou Flatpak Package - Build Instructions

## Summary

✅ **Successfully created:**
- Flatpak manifest (`gitlab.openlyst.doudou.yml`)  
- Desktop entry file (`gitlab.openlyst.doudou.desktop`)
- AppStream metadata (`gitlab.openlyst.doudou.metainfo.xml`)

✅ **Working features:**
- Downloads and unpacks the GitLab CI artifacts correctly
- Installs all Flutter bundle files in proper structure
- Creates wrapper script to run from correct directory
- Fixes Flutter AOT engine path issues
- Desktop integration (icon, menu entry)
- Proper permissions for Jellyfin connectivity

⚠️ **Known issue:**
- Media playback requires `libmpv` which needs to be added to the Flatpak build
- App launches but media playback won't work until this is resolved

## Overview
This repository contains the Flatpak packaging for Doudou v6.0.0, a beautiful Jellyfin music player built with Flutter.

## Files Created

### 1. `gitlab.openlyst.doudou.yml` - Flatpak Manifest
The main manifest file that describes how to build the Flatpak package.

**Key features:**
- Downloads the prebuilt Linux bundle from GitLab CI artifacts
- Uses Freedesktop Platform 24.08 runtime
- Includes all necessary permissions for audio playback and Jellyfin connectivity
- Creates a wrapper script to set proper environment variables

### 2. `gitlab.openlyst.doudou.desktop` - Desktop Entry
Standard desktop entry file for application menu integration.

**Provides:**
- Application name, icon, and description
- Proper categorization (AudioVideo → Audio → Player)
- MIME type associations for audio files
- Keyboard and keywords for search

### 3. `gitlab.openlyst.doudou.metainfo.xml` - AppStream Metadata
AppData/AppStream metadata for software centers.

**Contains:**
- Detailed app description and features
- Version 6.0.0 release information
- Developer information and links
- OARS content rating
- License information (GPL-3.0-or-later)

## Build Instructions

### Prerequisites & Known Limitations

**Current Status:** This Flatpak currently requires `libmpv` to be available on the host system, but due to Flatpak sandboxing, it cannot access host libraries properly. 

**For production/Flathub submission**, the manifest needs to be updated to either:
1. Build libmpv as part of the Flatpak (recommended for Flathub)
2. Use shared-modules for mpv
3. Package libmpv dependencies within the bundle

**Temporary workaround for local testing:**
- The app will fail to initialize media playback
- This doesn't prevent the app from launching, but media playback won't work

**TODO for Flathub submission:**
```yaml
# Add to modules section:
- name: libmpv
  # ... build instructions from shared-modules/libmpv
```

### Build the Flatpak
```bash
flatpak-builder --force-clean build-dir gitlab.openlyst.doudou.yml
```

### Install locally for testing
```bash
flatpak-builder --user --install --force-clean build-dir gitlab.openlyst.doudou.yml
```

### Run the installed app
```bash
flatpak run gitlab.openlyst.doudou
```

**Note:** The wrapper script changes to `/app/lib/doudou` before executing the Flutter binary. This is necessary because Flutter's AOT engine expects to find the `data/` directory relative to the executable's location.

### Create a Flatpak bundle for distribution
```bash
flatpak-builder --repo=repo --force-clean build-dir gitlab.openlyst.doudou.yml
flatpak build-bundle repo doudou.flatpak gitlab.openlyst.doudou
```

## Architecture Details

### Application Structure
```
/app/
├── bin/
│   └── doudou (wrapper script that cd's to /app/lib/doudou)
├── lib/doudou/
│   ├── doudou (actual Flutter binary)
│   ├── lib/
│   │   ├── libflutter_linux_gtk.so
│   │   ├── libapp.so
│   │   └── libmedia_kit_libs_linux_plugin.so
│   └── data/
│       └── flutter_assets/...
├── share/
│   ├── applications/
│   │   └── gitlab.openlyst.doudou.desktop
│   ├── metainfo/
│   │   └── gitlab.openlyst.doudou.metainfo.xml
│   └── icons/hicolor/64x64/apps/
│       └── gitlab.openlyst.doudou.png
```

**Important:** The entire Flutter bundle is kept intact in `/app/lib/doudou/` because Flutter's AOT engine expects to find the `data/` directory in the same location as the executable. The wrapper script in `/app/bin/doudou` changes to this directory before launching the app.

### Permissions Explained
- `--share=network` - Connect to Jellyfin servers
- `--socket=wayland/x11` - Display the GUI
- `--socket=pulseaudio` - Audio playback
- `--filesystem=xdg-music:ro` - Read-only access to music folder
- `--filesystem=xdg-download` - Download offline music
- `--talk-name=org.freedesktop.Notifications` - Show notifications
- `--talk-name=org.freedesktop.secrets` - Store credentials securely
- `--device=dri` - Hardware acceleration

## Artifact Information

**Source:** GitLab CI Artifacts  
**Job:** `build_release_linux`  
**Commit:** `c392f5d1da3d5636b945b6d14637d80d1217476c`  
**SHA256:** `dd90557598c012bf37930306849b52a12e7d401ba754df33f078807823211af1`  
**URL:** https://gitlab.com/Openlyst/doudou/-/jobs/artifacts/c392f5d1da3d5636b945b6d14637d80d1217476c/download?job=build_release_linux

## Submission to Flathub

To submit this to Flathub:

1. Fork https://github.com/flathub/flathub
2. Create a new repository for your app
3. Copy these files to the repository
4. Submit a pull request to flathub/flathub

See: https://github.com/flathub/flathub/wiki/App-Submission

## Notes

- The wrapper script sets `LD_LIBRARY_PATH` to ensure Flutter libraries are found
- Debug symbols are automatically stripped and compressed
- AppStream validation passed successfully
- All three files (.yml, .desktop, .xml) are required for Flathub submission

## Version History

- **6.0.0** (2025-10-06) - Initial Flatpak release

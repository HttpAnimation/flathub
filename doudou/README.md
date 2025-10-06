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
### Permissions Explained
- `--share=network` - Connect to Jellyfin servers
- `--socket=wayland/x11` - Display the GUI
- `--socket=pulseaudio` - Audio playback
- `--filesystem=xdg-music:ro` - Read-only access to music folder
- `--filesystem=xdg-download` - Download offline music
- `--talk-name=org.freedesktop.Notifications` - Show notifications
- `--talk-name=org.freedesktop.secrets` - Store credentials securely
- `--device=dri` - Hardware acceleration
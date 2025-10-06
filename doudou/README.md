# Doudou Flatpak Package - Build Instructions

Based on: https://github.com/Merrit/flutter_flatpak_example

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
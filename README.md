# Plasma Notification Mirror

A small Wayland utility for KDE Plasma that mirrors desktop notification popups to one or more additional monitors.

Plasma normally shows a notification popup on only one monitor. Plasma Notification Mirror monitors accepted Freedesktop notification requests on the session D-Bus and draws lightweight, non-focus-stealing copies using KDE's `layer-shell-qt`.

## Features

- Zero-configuration two-monitor default: mirrors to the first non-primary monitor
- Optional multi-monitor modes
- Per-monitor notification stacking
- Dynamic popup height
- KDE-like compact layout and expiration bar
- Countdown pauses while the pointer is over a popup
- Manual close button
- Existing notifications compact when one is dismissed
- Application icon support via `app_icon`, `image-path`/`image_path`, and `desktop-entry` fallbacks
- Typed D-Bus monitoring: notifications are mirrored only after the notification daemon accepts them and returns their notification ID
- Handles notification replacement and `NotificationClosed` events
- Configurable width, lifetime, margins, stack gap, and maximum popup count
- User-level systemd service; no root process required

## Requirements

- KDE Plasma on Wayland
- Qt 6
- `layer-shell-qt`
- CMake
- `libsystemd` / sd-bus (used for the D-Bus monitoring API)

### Arch Linux / CachyOS

```bash
sudo pacman -S --needed cmake qt6-base layer-shell-qt systemd-libs pkgconf
```

## Build

```bash
git clone https://github.com/MysteriousAeon/plasma-notification-mirror.git
cd plasma-notification-mirror
cmake -S . -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build -j$(nproc)
```

## Install for the current user

```bash
mkdir -p ~/.local/bin ~/.config/systemd/user
cp build/plasma-notification-mirror ~/.local/bin/plasma-notification-mirror
cp systemd/plasma-notification-mirror.service ~/.config/systemd/user/
systemctl --user daemon-reload
systemctl --user enable --now plasma-notification-mirror.service
```

No configuration file is required. With two monitors, the first non-primary monitor is selected automatically.

## Configuration

Optional configuration file:

```text
~/.config/plasma-notification-mirror/config.ini
```

Start from the included example:

```bash
mkdir -p ~/.config/plasma-notification-mirror
cp config/config.example.ini ~/.config/plasma-notification-mirror/config.ini
systemctl --user restart plasma-notification-mirror.service
```

### Monitor modes

```ini
[General]
mode=secondary
```

Available values:

- `secondary` — first non-primary monitor; default and zero-config behavior
- `all-secondary` — every non-primary monitor
- `screens` — only the monitor names in `screens=`
- `primary` — primary monitor only
- `all` — every monitor

Example for selected monitors:

```ini
[General]
mode=screens
screens=DP-2,DP-3
```

Monitor names can be viewed with:

```bash
kscreen-doctor -o
```

Other settings:

```ini
[General]
lifetime_ms=5000
width=332
max_notifications=5
right_margin=18
bottom_margin=58
gap=12
```

## Testing

```bash
notify-send -i dialog-information "Mirror test" "This should appear on the configured mirror monitor."
```

## Updating a running user install

Replacing a currently executing binary directly can produce `Text file busy`. Use an atomic replacement:

```bash
cmake --build build -j$(nproc)
cp build/plasma-notification-mirror ~/.local/bin/plasma-notification-mirror.new
chmod +x ~/.local/bin/plasma-notification-mirror.new
mv -f ~/.local/bin/plasma-notification-mirror.new ~/.local/bin/plasma-notification-mirror
systemctl --user restart plasma-notification-mirror.service
```

## Uninstall

```bash
systemctl --user disable --now plasma-notification-mirror.service
rm -f ~/.config/systemd/user/plasma-notification-mirror.service
rm -f ~/.local/bin/plasma-notification-mirror
systemctl --user daemon-reload
```

The optional config directory can also be removed:

```bash
rm -rf ~/.config/plasma-notification-mirror
```

## Notes

This is intentionally a notification **mirror**, not a replacement notification daemon. It does not attempt to duplicate notification action buttons or Plasma's notification history. It does track daemon-side notification replacement and close events so the mirror stays aligned with the notification service.

Some applications provide icons differently. The program currently tries the standard `app_icon` field, `image-path`/`image_path` hints, desktop-entry metadata, and icon-theme fallbacks. Raw `image-data` pixel payloads are not decoded yet.

## License

MIT

# appimage-wrapper

Small helpers for launching versioned AppImage files and generating desktop entries that keep using the newest installed version.

## Why

Many AppImages update themselves from inside the application. After an update, the on-disk filename often changes because the version is part of the filename, so a desktop entry that points at one exact AppImage path can stop working.

`appimage-wrapper` keeps desktop entries stable by launching an app by name, finding the highest `APP-*.AppImage` version available in `PATH`, and optionally deleting older versions after a successful selection.

Many AppImages are Electron or Chromium applications, but they do not all honor Arch-style user flag files such as `~/.config/<app>-flags.conf`, `~/.config/electron-flags.conf`, or app-specific `user-flags.conf` files. The wrapper provides a consistent place to load per-app flags and environment variables before launching the AppImage.

## Commands

### `appimage-wrapper`

```bash
appimage-wrapper [--cleanup-old] [--flags-file FILE] [--no-flags] [--env-file FILE] [--no-env] APP [args...]
```

`appimage-wrapper` searches `PATH` for files named:

```text
APP-VERSION.AppImage
```

It selects the highest version with `sort -V` and runs it. Extra arguments after `APP` are passed to the AppImage unchanged.

Examples:

```bash
appimage-wrapper MQTTX
appimage-wrapper --cleanup-old MQTTX --enable-features=WaylandWindowDecorations
APPIMAGE_WRAPPER_DIR=~/.local/bin appimage-wrapper minerU
```

`--cleanup-old` removes older matching `APP-*.AppImage` files after the newest file is selected and before it is launched.

### Flags files

By default, flags are read in this order:

```text
~/.config/APP-flags.conf
~/.config/APP/flags.conf
~/.config/APP/user-flags.conf
```

Each non-empty, non-comment line is passed as one argument to the AppImage.

```text
--enable-features=WaylandWindowDecorations
--ozone-platform-hint=auto
```

Use `--flags-file FILE` to read only explicitly listed files. It can be repeated. Use `--no-flags` to disable flags file loading.

### Env files

By default, environment files are read in this order:

```text
~/.config/APP.env
~/.config/APP/.env
~/.config/APP/user.env
```

Supported lines are `KEY=VALUE` and `export KEY=VALUE`. Empty lines and `#` comments are ignored. Files are parsed, not sourced.

Use `--env-file FILE` to read only explicitly listed files. It can be repeated. Use `--no-env` to disable env file loading.

### `appimage-desktop`

```bash
appimage-desktop [options] APP [-- app args...]
```

Creates a user desktop entry in:

```text
~/.local/share/applications/APP.desktop
```

Common options:

```bash
appimage-desktop MQTTX \
  --name MQTTX \
  --comment "MQTT Client Tool (AppImage)" \
  --icon mqttx \
  --icon-install /path/to/mqttx.png \
  --categories "Network;Development;" \
  -- --enable-features=WaylandWindowDecorations --ozone-platform-hint=auto
```

Defaults:

- If the newest matching AppImage contains a desktop entry, its metadata is used as the generated desktop entry template.
- `Exec=` is always replaced with the generated `appimage-wrapper` command.
- Explicit options such as `--name`, `--comment`, `--icon`, `--categories`, `--startup-wm-class`, and `--terminal` override the bundled desktop entry.

If no bundled desktop entry is available, built-in defaults are used:

- `Name=APP`
- `Comment=AppImage application`
- `Categories=Utility;`
- `StartupWMClass=APP`
- `Terminal=false`
- `Icon=` is the sanitized desktop id, unless an icon name is provided or an AppImage icon is extracted

`--icon NAME` sets the icon name written to the desktop entry. `--icon-install FILE` installs a local `png`, `svg`, `svgz`, or `xpm` icon into the user hicolor theme. When both are used, the file is installed under `NAME`; when only `--icon-install` is used, it is installed under the sanitized desktop id.

When neither icon option is provided, `appimage-desktop` tries to install the icon referenced by the bundled desktop entry from the newest matching `APP-*.AppImage` found through `APPIMAGE_WRAPPER_DIR` or `PATH`. Metadata and icon extraction are best effort; if they fail, the desktop entry is still generated with built-in defaults.

## Installation

From a source checkout:

```bash
install -Dm755 bin/appimage-wrapper ~/.local/bin/appimage-wrapper
install -Dm755 bin/appimage-desktop ~/.local/bin/appimage-desktop
install -Dm644 bash/appimage-wrapper ~/.local/share/bash-completion/completions/appimage-wrapper
install -Dm644 bash/appimage-desktop ~/.local/share/bash-completion/completions/appimage-desktop
install -Dm644 zsh/_appimage-wrapper ~/.zfunc/_appimage-wrapper
install -Dm644 zsh/_appimage-desktop ~/.zfunc/_appimage-desktop
```

Or build the Arch package:

```bash
makepkg -si
```

## Completion

System package installs:

- Bash completions to `/usr/share/bash-completion/completions`
- Zsh completions to `/usr/share/zsh/site-functions`

Reload Bash completions by opening a new shell or sourcing the installed completion file.

Reload Zsh completions with:

```zsh
unfunction _appimage-wrapper _appimage-desktop 2>/dev/null
autoload -Uz _appimage-wrapper _appimage-desktop
```

## Validation

```bash
bash -n bin/appimage-wrapper bin/appimage-desktop bash/appimage-wrapper bash/appimage-desktop
zsh -n zsh/_appimage-wrapper zsh/_appimage-desktop
```

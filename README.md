# clipboard-linux

A Windows-style clipboard history for Linux desktops (tested on Ubuntu 22.04,
GNOME on Wayland). Press **Super+V**, pick an entry from a small dialog, and it
is copied back to your clipboard — just like `Win+V` on Windows.

- Rolling history of copied **text** (up to 100 entries, 256 KB each)
- **Pin** entries you want to keep — pinned entries sort to the top
- **Restarting the service (or rebooting) erases everything except pinned
  entries**
- Search-as-you-type, keyboard navigation, right-click menu
- Runs as a systemd **user** service, starts at login

## Requirements

- Linux with X11 or Wayland (GNOME recommended)
- Python 3 with PyGObject/GTK 3 (`python3-gi`, `gir1.2-gtk-3.0` — installed by
  default on Ubuntu)
- `xclip` — clipboard watching (`apt install xclip`)
- `wl-clipboard` (`wl-copy`) — setting the clipboard from the dialog on Wayland
  (`apt install wl-clipboard`)

No Python packages are needed; everything uses the standard library + system
GTK.

## Install

```sh
./clipboard-linux install
```

This copies the script to `~/.local/bin/clipboard-linux`, installs the systemd
user unit `~/.config/systemd/user/clipboard-linux.service`, and enables and
starts it:

```sh
systemctl --user status clipboard-linux
```

## Bind Super+V (manual, GNOME)

Wayland does not let applications grab global keys themselves, so GNOME must
own the binding:

1. Open **GNOME Settings → Keyboard → Keyboard Shortcuts → Custom Shortcuts**
2. Click **+** to add one:
   - **Name:** `Clipboard History`
   - **Command:** `~/.local/bin/clipboard-linux toggle` — use the absolute
     path, e.g. `/home/YOU/.local/bin/clipboard-linux toggle`
   - **Key:** press `Super+V`
3. Save.

`clipboard-linux shortcut-hint` prints these instructions (and warns if
Super+V is already taken by another custom shortcut).

## Usage

Press **Super+V** (or run `clipboard-linux toggle`):

| Action | How |
| --- | --- |
| Copy an entry | single click, or `Enter` |
| Navigate | `Up` / `Down` (typing focuses the search box) |
| Search | just type; `Enter` copies the first match |
| Pin / unpin | right-click → **Pin**/**Unpin**, or `Ctrl+P` |
| Delete an entry | right-click → **Delete**, or `Del` |
| Close the dialog | `Esc`, or click elsewhere |

Picking an entry copies it and moves it to the top of the history (like the
Windows clipboard history does), then the dialog closes.

## How it works

- The daemon polls the X11 clipboard (via XWayland) twice a second. On GNOME
  the shell bridges the X11 and Wayland clipboards, so copies from both
  Wayland and X11 applications are captured. Image-only clipboards are
  ignored.
- History lives in `~/.local/share/clipboard-linux/history.json`. On daemon
  **startup** only `pinned` entries are loaded — everything else is erased.
- The dialog is a borderless, always-on-top GTK window placed near the mouse
  cursor (this is why the daemon runs with `GDK_BACKEND=x11`).

## Uninstall

```sh
clipboard-linux uninstall          # stop + remove the service
clipboard-linux uninstall --purge  # also remove ~/.local/bin/clipboard-linux
                                   # and ~/.local/share/clipboard-linux
```

## Files

| Path | Purpose |
| --- | --- |
| `~/.local/bin/clipboard-linux` | installed executable |
| `~/.config/systemd/user/clipboard-linux.service` | user service |
| `~/.local/share/clipboard-linux/history.json` | history + pins |

## Limitations

- Text only in v1 (images/other MIME types are skipped).
- The history is wiped (except pins) on every daemon restart, by design.
- On pure Wayland sessions without XWayland the dialog cannot be positioned
  near the cursor; it then opens where the compositor decides.

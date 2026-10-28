# deskflow-hyprland-clipboard

Clipboard sharing for a [Deskflow](https://github.com/deskflow/deskflow) **server running on Hyprland**.

Deskflow 1.27 on Wayland reaches the clipboard only through the xdg Clipboard portal.
`xdg-desktop-portal-hyprland` doesn't implement that portal, so on Hyprland the mouse and keyboard
cross over fine but copy/paste silently does nothing, in either direction.

This patch adds a fallback: when the portal session reports no clipboard, Deskflow talks the
[`ext-data-control-v1`](https://wayland.app/protocols/ext-data-control-v1) Wayland protocol
directly. Hyprland supports it (so do sway and other wlroots compositors).

> **This is an unofficial patch.** The Deskflow maintainers have said they only support the portal
> clipboard (see [#10315](https://github.com/deskflow/deskflow/issues/10315)), so please don't
> report problems with this build to the Deskflow tracker. Open an issue here instead.

## What it does

- **Server → client:** when the cursor crosses to a client, the server's current Wayland clipboard
  is read and sent over.
- **Client → server:** when a client's clipboard arrives, it's offered as the Wayland selection, so
  you can paste it on the server.
- **Formats:** UTF-8 text, HTML, and images (PNG on Wayland ↔ bitmap on the wire). Format conversion
  reuses Deskflow's own `PortalClipboard` helpers.
- **No change elsewhere:** the fallback only starts when the portal clipboard isn't available, so
  GNOME/KDE behave exactly as before. If the compositor lacks `ext-data-control-v1` as well, Deskflow
  logs a warning and carries on without a clipboard, as it does today.
- All Wayland calls run on one dedicated thread with its own display connection.

When it's active, the server log shows:

```
INFO: using wayland data-control for clipboard sharing
```

## Scope

- Tested: Hyprland 0.56 server (Arch Linux / Omarchy 4.0.6, xdg-desktop-portal-hyprland 1.4.1)
  with a Windows 11 client, text in both directions.
- Server role only. Wayland *clients* on Hyprland need the RemoteDesktop portal, which Hyprland
  doesn't provide yet; this patch doesn't change that.
- Regular clipboard only, not the primary (middle-click) selection.

## Install (Arch Linux)

`PKGBUILD` is Arch's official `deskflow` recipe plus this patch (pkgrel `2.1`, adds `wayland` and
`wayland-protocols`).

```bash
git clone https://github.com/gogetwoo/deskflow-hyprland-clipboard
cd deskflow-hyprland-clipboard
makepkg -si
```

Then restart the Deskflow server (Stop/Start in the app).

### Keep it from being replaced

A regular `pacman -Syu` will replace this build with the stock package once Arch ships a newer
deskflow. To hold it, add this under `[options]` in `/etc/pacman.conf`:

```
IgnorePkg = deskflow
```

When a new Deskflow version comes out, bump `pkgver`, run `makepkg -g` for checksums, rebuild,
and only then lift the hold.

## Other distros / building from source

The patch applies to Deskflow tag `v1.27.0`:

```bash
git clone --branch v1.27.0 https://github.com/deskflow/deskflow
cd deskflow
git apply ../0001-wayland-data-control-clipboard.patch
cmake -B build -G Ninja -DCMAKE_BUILD_TYPE=Release && cmake --build build
```

Build needs `wayland-client`, `wayland-protocols` ≥ 1.39 and `wayland-scanner`, on top of Deskflow's
usual dependencies (libei, libportal with clipboard support, Qt 6).

## The proper fix

The real long-term fix is a Clipboard portal in `xdg-desktop-portal-hyprland`, which first needs
RemoteDesktop portal support
([hyprwm/xdg-desktop-portal-hyprland#252](https://github.com/hyprwm/xdg-desktop-portal-hyprland/issues/252)).
Once that lands, stock Deskflow should work and this patch can be retired.

## Credits and license

- Patch written with Claude Code (Anthropic's AI coding assistant), then reviewed, built and tested on real hardware.
- Deskflow is © its contributors, GPL-2.0-only WITH LicenseRef-OpenSSL-Exception. This patch is
  under the same terms; see `LICENSE` (GPL-2.0).
- `PKGBUILD` is derived from Arch Linux's
  [deskflow packaging](https://gitlab.archlinux.org/archlinux/packaging/packages/deskflow).

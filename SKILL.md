---
name: omarchy-drawer-icon
description: >
  Recolour the Omarchy dock's app-drawer button — the grid icon that opens the
  app grid — so it matches the current theme, by resolving the theme's accent
  colour, rewriting the drawer PNG beside the dock plugin, and reloading the
  shell. Use when asked to change, set, tint, theme or match the dock's
  app-drawer / app-launcher icon colour, to make the dock follow the theme on
  theme change, or to install a custom app-drawer button. Triggers: drawer
  icon, app drawer icon, app-launcher icon, dock icon colour, tint the drawer,
  make the dock match the theme, omarchy-drawer-icon. Not for changing
  application icons, the application grid itself, or the bar.
---

# Recolouring the dock's app-drawer icon

The dock's app-drawer button is the first slot of the dock: `component AppsSlot`
in a dock plugin's `DockWindow.qml`. When it draws a custom icon it reads
`drawer-icon.png` from beside that file, and **that PNG carries its own colour**.

`omarchy-drawer-icon` is the tool for that colour. It resolves a colour from the
current theme, rewrites the PNG to it, and reloads the shell.

## Run it

```bash
omarchy-drawer-icon                 # theme accent, then restart the shell
omarchy-drawer-icon --no-reload     # the dock re-reads the PNG on its own
omarchy-drawer-icon -k color4       # a different theme key
omarchy-drawer-icon -c '#ff00aa'    # a literal colour
omarchy-drawer-icon -p gabriel.dock # pick the plugin explicitly
omarchy-drawer-icon -i ~/my-icon.png
```

It prints the colour, the theme name and the file it rewrote. It is idempotent:
running it twice with the same colour changes nothing.

Options: `-c/--color`, `-k/--key` (default `accent`), `-f/--fallback`
(default `color4`), `-i/--icon`, `-p/--plugin`, `-n/--no-reload`, `-q/--quiet`,
`-h/--help`.

## Keep it on the theme

A theme change rewrites the palette but leaves the PNG alone. A `theme-set` hook
closes that gap, so the drawer follows every theme:

```bash
omarchy hook install theme-set ~/.agents/skills/omarchy-drawer-icon/hooks/10-drawer-icon
```

The hook runs `omarchy-drawer-icon --quiet --no-reload` and does nothing when no
plugin has a `drawer-icon.png`. Remove it with
`rm ~/.config/omarchy/hooks/theme-set.d/10-drawer-icon`.

## What the QML has to look like

The script only writes the PNG, so the plugin must show it unmodified — a plain
`Image` in `AppsSlot`:

```qml
Image {
  x: window.iconInset
  y: window.iconInset
  width: window.iconSize
  height: window.iconSize
  source: Qt.resolvedUrl("drawer-icon.png")
  sourceSize: Qt.size(Math.round(window.iconSize * 2), Math.round(window.iconSize * 2))
  fillMode: Image.PreserveAspectFit
  asynchronous: false
  smooth: true
  mipmap: true
  scale: appsSlot.hovered ? 1.15 : 1
  transformOrigin: window.popOrigin

  Behavior on scale { NumberAnimation { duration: 120; easing.type: Easing.OutCubic } }
}
```

Do **not** also colourise it in QML. `MultiEffect.colorization` multiplies the
colourisation colour by the source's luminance, so a tinted PNG comes out
muddy: the accent `#7c3aed` tinted to white lands on `#626262`, its own
brightness in grey, not `#7c3aed`. Pick one mechanism: colour in the PNG with
this script, or tint in QML, never both.

The artwork should be a one-colour silhouette on transparency. The script keeps
the alpha channel and replaces every pixel's RGB, so a multi-colour image is
flattened to a single colour; keep a white master copy if a source is needed
again.

## How it works

- `omarchy-theme-color <key> [fallback]` resolves the colour from the current
  theme's `colors.toml`, walking the same alias/fallback cascade the generated
  configs use. Read it with `omarchy-theme-color --all` to see the palette.
- `magick <icon> -strip -fill <color> -colorize 100% PNG32:<tmp>` sets every
  pixel to the colour and leaves alpha alone; `-strip` drops ImageMagick's
  date/tIME metadata so one colour always produces the same bytes.
- The alpha channel is compared before the swap and the rewrite is refused if it
  moved, so the artwork's shape cannot be damaged.
- The new file is written beside the old one and moved into place, so an
  interrupted run cannot leave a half-written icon.
- If the icon already carries the colour, the file is left untouched — same
  bytes, same mtime — so a `theme-set` hook re-run gives the dock nothing to
  reload.
- `omarchy restart shell` makes the change unmistakable. The dock usually
  re-reads a changed PNG by itself, so `--no-reload` is fine — and is what the
  theme hook uses, since a theme change must not fight the intro animation.
  A non-zero exit from the restart (the shell can report its notification
  service not being ready) does not mean the rewrite failed.

## Verify

The dock renders on floating workspaces. Check the icon is actually the expected
colour rather than trusting the script's exit code:

```bash
omarchy dock autohide off
grim -o eDP-1 /tmp/dock.png
omarchy dock autohide on

# the most common non-background colour in the drawer slot should be the accent
magick /tmp/dock.png -crop 74x66+878+1196 +repage -depth 8 -format %c histogram:info: |
  sort -rn | head -5
```

Coordinates differ per monitor and scale; find the slot from the dock's layer
geometry instead of copying them (`hyprctl layers`, the `omarchy-dock` surface).
Remember to restore `omarchy dock autohide on` if it was on.

## Requirements

- ImageMagick 7 (`magick`).
- A dock plugin that draws `drawer-icon.png` — clone the built-in one first so an
  update cannot overwrite the change:

  ```bash
  omarchy plugin clone omarchy.dock     # -> ~/.config/omarchy/plugins/<you>.dock
  omarchy plugin disable omarchy.dock
  omarchy plugin enable <you>.dock
  omarchy restart shell
  ```

# omarchy-drawer-icon

Recolour the Omarchy dock's app-drawer button — the grid icon that opens the app
grid — so it matches the current theme: resolve the theme's accent colour, rewrite
the drawer PNG beside the dock plugin, and reload the shell.

This repo is an agent skill for [opencode](https://opencode.ai). `SKILL.md` is the
instruction sheet the agent reads; `omarchy-drawer-icon` is the command it drives.

## Use

```bash
omarchy-drawer-icon                 # theme accent, then restart the shell
omarchy-drawer-icon --no-reload     # the dock re-reads the PNG on its own
omarchy-drawer-icon -k color4       # a different theme key
omarchy-drawer-icon -c '#ff00aa'    # a literal colour
omarchy-drawer-icon --help
```

It prints the colour, the theme name and the file it rewrote. Running it twice
with the same colour changes nothing.

## Install

On a machine that does not have it yet:

```bash
git clone https://github.com/gbenselum/omarchy-drawer-icon ~/.agents/skills/omarchy-drawer-icon
ln -sfn ~/.agents/skills/omarchy-drawer-icon ~/.claude/skills/omarchy-drawer-icon
ln -sfn ~/.agents/skills/omarchy-drawer-icon/omarchy-drawer-icon ~/.local/bin/omarchy-drawer-icon
omarchy hook install theme-set ~/.agents/skills/omarchy-drawer-icon/hooks/10-drawer-icon
```

The last line makes the drawer follow every theme change. Skip it if you would
rather run the command by hand.

## Requirements

- ImageMagick 7 (`magick`).
- Omarchy, with a dock plugin that draws `drawer-icon.png` — see `SKILL.md` for
  the `AppsSlot` QML and how to clone the built-in dock so an update cannot
  overwrite your icon.

## Layout

| Path | What it is |
| --- | --- |
| `SKILL.md` | the skill: when to use it, the QML it expects, the gotchas |
| `omarchy-drawer-icon` | the script |
| `hooks/10-drawer-icon` | `theme-set` hook that keeps the drawer on the theme |

## Uninstall

```bash
rm ~/.config/omarchy/hooks/theme-set.d/10-drawer-icon
rm ~/.local/bin/omarchy-drawer-icon ~/.claude/skills/omarchy-drawer-icon
rm -r ~/.agents/skills/omarchy-drawer-icon
```

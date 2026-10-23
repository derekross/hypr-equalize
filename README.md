# hypr-equalize

Give every tiled window on the current Hyprland workspace an equal share of the screen.

Hyprland's dwindle layout is a binary tree, so `splitratio 1` only evens out a
single split. `hypr-equalize` rebuilds the workspace's rows and columns from
window geometry and evens out all of them:

- **Side-by-side windows** get equal widths.
- **Stacked windows** get equal heights.
- **A mixed grid** gets each row or column split evenly between the groups in
  it. Three columns of stacked windows become three equal columns, each split
  evenly top to bottom.

Focus goes back to the window you were on.

## Why it doesn't just resize each window

In a dwindle tree, resizing a window moves its parent split, and which way it
moves depends on where the window sits in the tree. In a `(A | B) | C` layout
(easy to get after a `togglesplit`), asking B to get wider actually shrinks it,
so setting each window to an absolute size can run away until one window takes
nearly the whole screen.

So `hypr-equalize`:

1. **Probes:** nudges each window by a few pixels and measures which divider
   actually moves, and in which direction. This also reaches dividers that no
   window sits directly beside, which `splitratio` can't touch.
2. **Converges:** moves every divider to its equal-share position and repeats
   until each one is within 2px.
3. **Restructures if needed:** if a single row or column has a divider that no
   resize moves (for example `(A | B) | (C | D)`), it hoists the first window to the
   root with `movetoroot`, then swaps the windows back into their original
   order.

`smart_resizing` is turned off while it runs and restored afterwards.

## Requirements

- Hyprland with the Lua config (`hl.dsp.*` dispatchers), as shipped by
  [Omarchy](https://omarchy.org/). Tested on Hyprland 0.56.2.
- The dwindle layout.
- Python 3 (standard library only).

## Install

```sh
curl -fsSL https://raw.githubusercontent.com/derekross/hypr-equalize/main/hypr-equalize \
  -o ~/.local/bin/hypr-equalize
chmod +x ~/.local/bin/hypr-equalize
```

Bind it in `~/.config/hypr/bindings.lua` (Omarchy):

```lua
o.bind("SUPER + SHIFT + J", "Equalize windows", os.getenv("HOME") .. "/.local/bin/hypr-equalize")
```

Run it with `--verbose` to print the detected layout and the resulting window sizes.

## License

MIT

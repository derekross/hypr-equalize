# hypr-equalize

Give every tiled window on the current Hyprland workspace an equal share of the screen.

![Six windows of different sizes snapping into an even 3x2 grid after one press of Super+Shift+J](https://raw.githubusercontent.com/derekross/hypr-equalize/main/demo.gif)

Hyprland's dwindle layout is a binary tree, so `splitratio 1` only evens out a
single split. `hypr-equalize` makes every tiled window the same size:

- **Side-by-side windows** get equal widths.
- **Stacked windows** get equal heights.
- **A grid** gets identical cells. If the current arrangement can't get there
  (see below), the windows are re-tiled as an even grid, keeping their
  reading order: left to right, top to bottom.

Focus goes back to the window you were on. Running it on a workspace that is
already even does nothing.

## Why it doesn't just resize each window

In a dwindle tree, resizing a window moves one of its ancestor splits, and
which one, and which way, depends on where the window sits in the tree. In a
`(A | B) | C` layout (easy to get after a `togglesplit`), asking B to get
wider actually shrinks it, so setting each window to an absolute size can run
away until one window takes nearly the whole screen. And when edges line up,
geometry alone can't tell a 2x2 block beside a column from four columns.

So `hypr-equalize`:

1. **Probes:** nudges each window by a few pixels. The windows that change are
   the two sides of the one split that moved, which gives the real tree and
   the window that controls each split.
2. **Converges:** moves every split to the spot where each window under it
   gets the same area, and repeats until each one is within 2px.
3. **Rebuilds if needed:** a keyboard resize only reaches a window's parent or
   grandparent split, so a split with a whole subtree on each side never
   moves. If any split can't be reached, or the tree is lopsided enough that
   equal areas still mean different shapes, every window but the first is
   floated and tiled back in with `preselect`, building even columns of
   stacked windows. A row stays a row and a stack stays a stack; anything
   else gets the grid closest to square cells.

`smart_resizing`, `smart_split`, `use_active_for_splits` and
`default_split_ratio` are set while it runs and restored afterwards.

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

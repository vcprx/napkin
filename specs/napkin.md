# Napkin

A browser-based freehand drawing board.

## Stack

Svelte 5 (runes) on plain Vite, TypeScript (strict), Tailwind CSS v4, Lucide icons (`@lucide/svelte`), HTML `<canvas>` 2D context, Pointer Events. Type checking with `svelte-check`. No tests.

| Script          | Purpose                    |
|-----------------|----------------------------|
| `npm run dev`   | Dev server                 |
| `npm run build` | Production build to `dist/`|
| `npm run check` | Type check                 |

## Structure

```
src/
  main.ts                  # mounts App
  app.css                  # @import 'tailwindcss'
  App.svelte               # owns brush state (color, width); composes Board + Toolbar
  Board.svelte             # full-window canvas: drawing, undo, resize, high-DPI
  Toolbar.svelte           # undo, color and line width pickers
  BrushStyleButton.svelte  # 36×36 button used for all toolbar buttons and swatches
```

`App` holds `color` and `width` as `$state`. `Toolbar` binds to them (`bind:color`, `bind:width`), and `Board` receives them as props. `Board` exports `undo` and `canUndo`; `App` reaches them through `bind:this` and passes them to `Toolbar` as `onundo` and `canUndo`.

## Features

### Drawing

- The canvas fills the whole window. The page does not scroll.
- Draw freehand with mouse (primary button only), touch, or pen.
- Strokes have round caps and joins.
- A tap without movement draws a dot whose diameter equals the line width.
- Only one stroke is drawn at a time. Other pointers (for example, a second finger) are ignored until it ends.
- The stroke continues while the pointer is held, even outside the window, and ends cleanly on release or cancel.
- Touch input draws instead of scrolling or zooming the page (`touch-action: none`).
- Fast movement stays smooth: all coalesced pointer events are used when the browser provides them.

### Undo

- Undo permanently removes the most recent stroke. History is unlimited.
- Undo is ignored while a stroke is in progress.

### Brush style

- **Color**, 5 neon presets, default Cyan:

  | Name    | Value     |
  |---------|-----------|
  | Cyan    | `#00e5ff` |
  | Magenta | `#ff2bd6` |
  | Lime    | `#39ff14` |
  | Yellow  | `#ffe600` |
  | Orange  | `#ff7a00` |

- **Line width**, 5 presets in CSS pixels, default 2: `1`, `2`, `4`, `8`, `16`.
- A stroke takes the color and width selected when it starts. Changing the selection affects only new strokes. Existing strokes and a stroke in progress keep their style.

### Toolbar

- Floats at the bottom center of the window, over the canvas.
- Buttons side by side: Undo (Lucide `undo-2` icon, `zinc-300`), Color (a filled circle in the current color), Line width (a bar at the current thickness, in the current color).
- Undo is disabled (dimmed, no hover background) when there are no strokes.
- Clicking a trigger toggles a popover above it, listing the 5 presets in a column with the same swatch style. Only one popover is open at a time.
- Picking a preset selects it and closes the popover.
- While a popover is open, pressing anywhere outside the toolbar closes it, and that press does not draw. Escape also closes it.
- Dark surface (`zinc-900`) with a faint white ring, so it stands out from the background.
- Buttons are 36×36 px with a lighter gray (`zinc-800`) hover background.
- Clicking the toolbar never draws on the canvas.

### Accessibility

- Undo, triggers and swatches are `<button>` elements, reachable with Tab and operable with Enter and Space, with a visible focus ring. A disabled Undo is skipped by Tab.
- Undo has `aria-label="Undo"`.

### Display

- Dark theme: the page behind the transparent canvas is near-black (`zinc-950`).
- Lines are crisp on high-DPI screens: the canvas backing store is sized to `CSS size × devicePixelRatio`.
- Resizing the window keeps every stroke at the same CSS-pixel position, including a stroke in progress.
- A change of `devicePixelRatio` without a resize (for example, moving the window to another monitor) also resizes the backing store.

## How drawing works

- Strokes are kept as vector data (`{ color, width, points }`) in a list local to `Board`. A stroke is added to the list when it starts.
- The list is `$state.raw` and only ever reassigned, so points pushed during a stroke are not tracked.
- While drawing, only the new segment since the last pointer event is drawn onto the single canvas. Nothing is redrawn during a stroke.
- Undo drops the last stroke from the list, clears the canvas, and redraws every remaining stroke.
- On resize or a `devicePixelRatio` change, the backing store is resized, which clears it, and every stroke is redrawn from the list. The resize effect reads the stroke list untracked, so adding a stroke does not trigger it.
- A single-point stroke is drawn as a 0.01 px segment, because canvas skips zero-length segments and would otherwise draw nothing.

## Limitations

- Each stroke is drawn segment by segment, so a future semi-transparent style would show darker overlaps at the joints.
- Strokes live only in memory. Reloading the page clears the drawing.
- No keyboard shortcut for undo.
- Only Undo has an accessible name. The other toolbar buttons have no `aria-label` (`Red`, `4 px`, …), no `role="group"` for Color and Line width, no `aria-expanded` on triggers, no `aria-pressed` on swatches. Screen readers announce unlabeled buttons.
- Popovers do not mark the selected preset; the current choice is only visible on the trigger.
- Not built: redo, eraser, shapes, text, selection, custom colors or widths, pan/zoom, saving, exporting, collaboration.

# Layer Manager: visual spec

Source: artboard **Layer Manager** (`mockup/Layers.dc.html`, 1440 × 860). Rendered states:
`states/*.html` here, PNGs (2×) on the `pr-assets` branch. Units are egui points; colours are
CADCraft dark-theme tokens (light theme uses the same token names). One accent colour only
(`accent`); everything else is grey. Layer swatches show layer colours (content).

| Hex in mockup | Token |
|---|---|
| `#3b4453` | `chrome` (window body) |
| `#313844` | `chrome_dark` (title bar, footer, segmented track) |
| `#363e4b` | side panes (sidebar, inspector): `chrome` darkened like Publish's inspector |
| `#2b313b` | `list_bg` (table, fields) |
| `#232930` | `border` |
| `#505a6d` | `control` (buttons, selected segment, selected filter) |
| `#3d4a5c` + 1 pt `#3d8bfd` inset | `tab_active` + `accent` (selected row) |
| `#2f3846` | override row tint (`list_row`) |
| `#eef1f4` / `#d7dbe0` / `#a3aab4` / `#878d96` / `#6b7586` | `text` bright / `text` / `text_dim` / `text_faint` / off-icon |
| `#3d8bfd` | `accent` (checkboxes, Save Filter, override dots, focus ring) |
| `#1f242b` + `#3a4250` stroke | popup menu fill and stroke (menu style of the menu bar spec) |

## Window

| Property | Value |
|---|---|
| Size | 1360 × 772 default, min 980 × 560, resizable, fits the screen like Publish |
| Title bar | 30, `chrome_dark`, "Layers" centred 13 pt, close 24 |
| Toolbar | 42, padding 0 12, bottom rule `border` |
| Body | grid 210 · flex · 300 |
| Footer | 34, `chrome_dark`, top rule: "Current layer: **A-WALL**" · divider · "All layers: 24 of 24 layers" |

## Toolbar (left to right)

- **New Layer** button (26 high, `control`, glyph + label), ⌥N / Alt-N.
- Icon buttons 30 × 26 `control`: new layer frozen in every existing viewport · delete (unused
  only) · make current.
- Divider 1 × 18.
- **State ▾** button (26, `chrome_dark` fill, `border` stroke): "State" faint + state name bright.
- Spacer.
- Segmented **Drawing | Viewport: <name>** (28 high track, segments 22, radius 7/5). Disabled at
  45% opacity when no layout viewport is active.
- Search field 230 × 28 with magnifier, placeholder "Search layers  (A-* or *wall*)".
- Gear 30 × 26 → Settings popover.

## Filters sidebar (210, side-pane fill)

- Header 32: "FILTERS" (11 pt 600, letter-spacing 0.08 em, `text_faint`) · "+" 24 × 22.
- Items 26 high, radius 5, glyph 13 + name 12.5 + count 11.5 tabular; selected = `control` fill.
- Built-ins: All layers · Used · From references · Viewport overrides (disabled outside a
  viewport, hover explains). Then headings "Rule filters" and "Groups" (11 pt `text_faint`).
- Bottom row: checkbox "Invert filter" · "⌘I".
- "+" menu (250 wide): Rule Filter… / Group / Group from Selected Objects, each with an 11.5 pt
  second line.

## Table

| Part | Spec |
|---|---|
| Frame | margin 10 12 0, `list_bg`, `border`, radius 4, scrolls |
| Columns | 16 · minmax(150, 1.5fr) · 30 · 30 · 30 · 30 · 108 · 118 · 92 · 56 · minmax(80, 1fr) · 26, gap 6, padding 0 8 |
| Header | 24, sticky, 11.5 pt 500 `text_faint`: · Name ↑ · On · Frz (VP in viewport scope) · Lock · Plot · Color · Linetype · Lineweight · Transp. · Description · ⋯ |
| Viewport scope headers | Color · here, Linetype · here, Lineweight · here |
| Row | 22, 12.5 pt; current = check glyph; name (reference part `file | ` dimmed) |
| Toggles | 26 × 20 buttons, 14 pt glyphs; on `#cfd4db`, off `#6b7586`; VP freeze on = accent-tinted |
| Colour | 11 × 11 swatch + name (index or colour name) |
| Linetype | 22 × 6 preview + name |
| Lineweight | bar 16 wide, height ∝ weight + value |
| Transparency | right-aligned number |
| Off or frozen row | text `text_faint` |
| Reference row | text `text_dim`; lock and plot disabled |
| Selected row | `tab_active` + 1 pt accent inset |
| Overridden cell | bright text + 5 pt accent dot; row tint `list_row` when "Mark viewport overrides" |
| Column menu ⋯ | Fit Columns to Content · Spread Columns to Fill · checkboxes Description, Plot style, Frozen in new viewports, Mark viewport overrides |
| Hint line | 30 high, 12 pt `text_faint`: "Click to select · double-click to make current · ⇧/⌘-click for several · drag rows onto a group" (viewport scope: "Changes here apply to the active viewport only. Marked cells differ from the drawing.") |
| Empty | "No layers match. Clear the search" (link) |

## Inspector (300, side-pane fill, padding 12 14, gap 12)

- Swatch 14 + name field 28 (600 14 pt) for rename.
- "38 objects" (`text_dim`) · spacer · **Make Current** or pill "Current layer".
- Section PROPERTIES (IN THIS VIEWPORT in viewport scope): grid 92 / rest — Color (swatch +
  name ▾), Linetype (48 × 8 preview + name ▾), Lineweight ▾, Transparency (slider 0–90 + "0%").
- Linetype dropdown: spans label + field (272 wide), max 420 high, scrolls. Headings IN THIS
  DRAWING / STANDARD SET; rows 28: 64 × 10 preview · name · ✓ on the current; footer "Load from
  File…" and note "Picking one from the standard set loads it into the drawing."
- Viewport scope box: "Differs here: color, lineweight" · Remove (disabled when nothing differs).
- PLOTTING: Plot style ("By color (table has no named styles)" or a named-style ▾) · Plot this
  layer · Frozen in new viewports.
- DESCRIPTION textarea 2 rows.
- OBJECTS ON THIS LAYER: Select · Isolate (↔ Show All Layers) · Merge Into… · Add to Group….

## Rule filter editor (replaces the inspector)

"NEW RULE FILTER" · Name field · segmented Match all rules | Match any rule · rule rows
(property ▾ 82 · test ▾ 76 · value · ✕) · "+ Add rule" · result box "3 layers match" · note "The
list shows the matches while you edit. A layer that stops matching leaves the filter by itself."
· Cancel / **Save Filter** (accent). The table shows the matches; footer reads "Rule preview: 3
of 24 layers".

## State menu (300 wide)

RESTORE heading · rows (● on the current state, name, note) · divider · Save Current As… ·
Import… · Export Selected… · Rename or Delete… · divider · note "A state remembers on/off,
freeze, lock, plot, colours, linetypes and viewport overrides. Restoring changes only those."

## Settings popover (320 wide, under the gear)

SETTINGS · Tell me when new layers come in with a reference · Apply the filter to the layer list
in the toolbar · Undo layer changes one by one · Mark viewport overrides in the table · Done.
(Lock fading: open decision in TASK.md.)

## States

| File | Shows |
|---|---|
| 01-drawing | Default: All layers, A-DOOR selected, Drawing scope |
| 02-viewport-overrides | Viewport scope; marked cells; inspector "Differs here" |
| 03-viewport-overrides-filter | Viewport overrides filter: 3 layers |
| 04-linetypes | Linetype dropdown open (20 linetypes) |
| 05-rule-filter-editor | New rule filter "S- and Locked", live preview 3 layers |
| 06-rule-filter-applied | Saved rule filter "Floor (red or cyan)" |
| 07-group-inverted | Group "Locked layers" with Invert filter |
| 08-states-menu | State ▾ menu |
| 09-new-filter-menu | "+" menu |
| 10-columns-menu | Column menu |
| 11-search | Search `PV-*` |
| 12-no-match | Empty search result |
| 13-model-space | Model space: viewport switch disabled |
| 14-xref-layer | From references filter, reference layer selected |
| 15-settings | Settings popover |

Sample layer names are invented for the mock-up; real data comes from the drawing.

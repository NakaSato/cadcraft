# Claude Code sequence: Layer Manager

Paste one step at a time. Wait for each to finish and for `cargo` to be green where it says so.
Each phase ends in a PR to `NakaSato/cadcraft` `main` from `feat/layer-manager` (rebase on
`main` first). Screenshots for PR descriptions go to the `pr-assets` branch.

## Phase 1: engine and document (PR 1)

**Step 1: orient and plan.** Paste `PROMPT.md`.

**Step 2: layer filters**
```
Implement IMPLEMENTATION.md §2.1: LayerFilter / LayerRule in cadcraft_doc with pure matching
and unit tests (truth table for every property and test, Match all/any, case), Drawing.
layer_filters, and the layerfilter.* commands plus layer.list {filter, invert, search}.
Rename, delete and merge of layers keep groups right. Persist as we agreed in step 1, with a
DXF round-trip test. Run cargo test -p cadcraft-doc -p cadcraft-engine.
```

**Step 3: viewport overrides**
```
Implement IMPLEMENTATION.md §2.2: linetype, lineweight and transparency overrides on Viewport,
vplayer linetypes/lineweights/transparency/clear, the renderer using all four overrides (colour
included) for ByLayer objects inside the viewport on screen and in plots, DXF persistence, and
audit cleanup. Tests: each override, clear, plot output differs only in that viewport,
round-trip. Show that cargo xtask layers still passes.
```

**Step 4: layer states and fading**
```
Implement §2.3 (states keep viewport overrides; save options; .cclayers export/import with
validation and hostile-file tests) and §2.4 (LAYLOCKFADECTL setting and screen-only fading of
locked layers). Then cargo xtask ci, commit, push, and open PR 1 "engine: layer filters, more
viewport overrides, layer state files, locked-layer fading" with a parameter table.
```

## Phase 2: the window, table and inspector (PR 2)

**Step 5: module and skeleton**
```
Split layers.rs into layers/{mod,table,filters,inspector,states}.rs as IMPLEMENTATION.md §3
says, keeping the public colour helpers' paths working. Build the window frame per
spec/layer-manager-spec.md: toolbar, three panes, footer, the Drawing | Viewport switch
(disabled outside a layout viewport). Draw the new glyphs in icons.rs. Screenshot and compare
with spec/states/01-drawing.html.
```

**Step 6: table**
```
Build the table: sticky header with sort, 22 pt custom-painted rows, toggles with accessible
labels, colour swatch, linetype preview (lt_sample), lineweight bar, transparency, description,
multi-select with ⇧/⌘, toggles applying to the whole selection in one undo step, double-click to
make current, F2 rename, keyboard from §3. Column menu (fit, spread, optional columns, mark
overrides). Screenshot 01 and 10.
```

**Step 7: inspector and viewport scope**
```
Build the inspector (name, objects, Make Current, Color, Linetype dropdown with "In this
drawing" / "Standard set" and drawn previews, Lineweight, Transparency, Plotting, Description,
Select / Isolate / Merge Into… / Add to Group…). Wire the Viewport scope: same columns edit the
active viewport through vplayer; overridden cells bright + accent dot; "Differs here: … Remove".
Screenshot 02, 03, 04.
```

**Step 8: strings, tests, PR 2**
```
Every string through tl!/tf! with uk.tsv rows placed as BRANCHES.md says. kittest tests in
crates/ui-egui/tests/layer_manager.rs for TASK criteria 2, 3, 8, 9, 11. cargo xtask ci, rebase on
main, push, put screenshots on pr-assets under layer-manager/, open PR 2.
```

## Phase 3: filters (PR 3)

**Step 9: sidebar and search**
```
Filters sidebar with counts (All, Used, From references, Viewport overrides, rule filters,
groups), wildcard search, Invert filter (⌘/Ctrl-I), empty state "No layers match. Clear the
search". Screenshot 07, 11, 12.
```

**Step 10: rule editor and groups**
```
New filter menu (Rule Filter…, Group, Group from Selected Objects). Rule editor in the
inspector pane with live preview and Save Filter / Cancel; context menu on a filter (Edit,
Rename, Delete, Duplicate); drag rows onto a group; remove from group. Tests for criteria 4–7.
cargo xtask ci, PR 3 with screenshots 05, 06, 09.
```

## Phase 4: states, references, settings, docs (PR 4)

**Step 11: states menu**
```
State ▾ menu: restore on click, Save Current As… (name, description, include viewports),
Import… / Export Selected… (.cclayers), Rename or Delete… window. Screenshot 08. Test criterion 10.
```

**Step 12: references, settings, delete guard**
```
Reference layers (file part dimmed, lock/plot disabled), delete guard with reasons, Settings
popover (as the owner decided for lock fading), model-space look (screenshot 13, 14, 15).
Update docs/ui-parity.md, docs/gaps.md, docs/file-format-parity.md with revision lines in a
separate last commit. cargo xtask ci, PR 4. Ask me whether docs/design/layer-manager/ stays.
```

## If something goes wrong

- **egui row painting is slow with thousands of layers** → paint only visible rows
  (`ScrollArea::show_rows`) and cache counts per frame.
- **Drag and drop between table and sidebar** → use egui's `dnd_drag_source` /
  `dnd_drop_zone` with a payload of layer names.
- **The DXF Reference doesn't describe a viewport override field** → our own XDATA, documented,
  never a guess at another program's format.
- **A rebase conflicts in uk.tsv** → keep both sides' rows; the block layout in BRANCHES.md keeps
  them apart.

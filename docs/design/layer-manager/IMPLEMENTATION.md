# IMPLEMENTATION: Layer Manager

Code map at `fc4e983` (Release 0.7.0, `main`). Line numbers drift; search by name.

## 1. What exists

### UI: `crates/ui-egui/src/layers.rs` (748 lines)

- `LpmState` (egui temp data `lpm_state`): `filter: u8` (0 All, 1 All Used Layers), `search`,
  `selected: Option<String>` (one row), `renaming`, `describing`, `sort_desc` (name only),
  `state_name`, `state_sel`.
- `dialog()`: reads `app.session.state()`, the active viewport (`dst.active_viewport()`), the
  standard linetype library not yet loaded (`cadcraft_doc::library::standard_linetypes()`), layer
  states (`d.layer_states`). Builds an `egui::Grid` with columns Status · Name · On · Freeze ·
  Lock · Plot · Color · Linetype · Lineweight · Transparency · Plot Style… + `VP Freeze`,
  `VP Color` inside a viewport + `New VP Freeze`, `Description`. Layer states are a combo with
  Restore / Delete / Save State in the toolbar.
- `row()`: one layer's cells; collects actions as `(command, json)` and runs them after the frame.
- Reusable pieces: `swatch`, `color_button`, `color_palette` (public, used elsewhere),
  `lw_sample`, `lt_sample` (draws a linetype from its pattern), `status_icon`, `natural_lower`,
  `used_layers`, `crate_wild` / `wild` (wildcards).
- Opened through `dialogs.rs` (`"layers" => crate::layers::dialog`).

### Engine: `crates/engine/src/cmd/layer.rs` (+ `layer/merge.rs`, `layer/prompts.rs`)

Commands: `layer`, `layer.new`, `layer.set {name, on?, frozen?, locked?, plot?, color?,
linetype?, lineweight?, transparency?, description?, newVpFreeze?, plotStyle?, newName?}`,
`layer.current`, `layer.delete`, `laymcur`, `laymch`, `laycur`, `layiso`, `layuniso`, `layfrz`,
`layoff`, `laylck`, `layulk`, `layon`, `laythw`, `layerp`, `laymrg`, `laydel`,
`layerstate.save {name}` / `.restore` / `.list` / `.delete` / `.rename`.

- `layerstate.save` copies `d.layers` (every `Layer` field) into `LayerState { name, layers }`;
  restore overwrites each saved layer whole. No options, no viewport overrides.

### Viewport overrides: `crates/engine/src/cmd/layout.rs` + `layout/vplayer.rs`

- `vplayer {handle?, freeze?: [layer], thaw?: [layer], colors?: {layer: color | null}}`.
- `Viewport` (`crates/doc/src/entity.rs`): `frozen_layers: Vec<String>`,
  `layer_colors: Vec<(String, Color)>`.
- **Check first:** at `fc4e983` nothing outside the UI, `audit.rs` and the `layout` JSON reads
  `layer_colors`. It looks like the viewport colour override is neither drawn by the renderer
  nor written to DXF. Confirm (grep `crates/render`, `crates/dxf`, `crates/io`), report, and fix
  it in phase 1 together with the new overrides.

### Document: `crates/doc/src/tables.rs`, `lib.rs`

- `Layer { name, color, linetype, lineweight, on, frozen, locked, plot, transparency (0–90),
  description, plot_style, vp_freeze_new }`.
- `Drawing.layer_states: Vec<LayerState>`; DXF round-trips layer states as foreign objects
  (#528, `docs/gaps.md` FF1).
- `Group` is the entity GROUP (objects), not a layer group; don't reuse it.
- Linetype library: `crates/doc/src/library.rs` (`standard_linetypes()`, 20 incl. Continuous;
  the design's dropdown lists exactly these).

### Settings: `crates/engine/src/lib.rs` `Settings`

Add `layer_lock_fade: u8` (LAYLOCKFADECTL, 0–90, default 50) next to
`plot_transparency_override`, with its sysvar plumbing like the others.

## 2. Engine and document additions (phase 1)

### 2.1 Layer filters

```rust
// crates/doc/src/tables.rs
pub struct LayerFilter { pub name: String, pub kind: LayerFilterKind }
pub enum LayerFilterKind {
    Rule { all: bool, rules: Vec<LayerRule> },
    Group { layers: Vec<String> },
}
pub struct LayerRule { pub property: RuleProperty, pub test: RuleTest, pub value: String }
// RuleProperty: Name, Color, Linetype, Lineweight, State, Used
// RuleTest: Contains, Is, IsNot, StartsWith
```

- `Drawing.layer_filters: Vec<LayerFilter>`; matching lives in `cadcraft_doc` (pure, tested):
  `fn matches(&self, layer: &Layer, used: bool) -> bool`.
- Commands (new file `crates/engine/src/cmd/layer/filters.rs`):
  `layerfilter.list` → `[{name, kind, count}]`,
  `layerfilter.save {name, all?, rules?: [{property, test, value}]} | {name, layers?: [..]}`,
  `layerfilter.delete {name}`, `layerfilter.rename {name, newName}`,
  `layerfilter.add {name, layers}` / `layerfilter.remove {name, layers}` (groups),
  `layerfilter.fromSelection {name}` (group of the selected objects' layers).
  Validate everything; names unique (case-insensitive); unknown keys rejected like other
  commands.
- `layer.list {filter?, invert?, search?}` (or extend the existing list): returns the layers the
  UI shows, so the UI and scripts agree on matching.
- Renaming or deleting a layer updates groups; layer merge (`laymrg`) too.
- Persistence: see TASK.md open decision. Default plan: an XRECORD per filter in a named
  dictionary `CADCRAFT_LAYER_FILTERS` written by our DXF writer; foreign layer filters keep
  round-tripping untouched. Round-trip test.

### 2.2 Viewport overrides

- `Viewport` gains `layer_linetypes: Vec<(String, String)>`,
  `layer_lineweights: Vec<(String, Lineweight)>`, `layer_transparency: Vec<(String, u8)>`.
- `vplayer` gains `linetypes?: {layer: name | null}`, `lineweights?: {layer: mm | "Default" |
  null}`, `transparency?: {layer: 0..90 | null}` and `clear?: [layer] | "all"`.
- Renderer: inside a viewport, a layer's colour, linetype, lineweight and transparency come from
  the override when present (ByLayer objects only), on screen and in plots.
- DXF: follow the published DXF Reference for viewport layer overrides; if it doesn't describe
  them precisely enough, store them in our own XDATA (app name `CADCRAFT`) on the viewport and
  say so in `docs/file-format-parity.md`.
- `audit.rs` drops overrides for layers that no longer exist (extend the existing
  `layer_colors` cleanup).

### 2.3 Layer states

- `LayerState` gains `viewports: Vec<(Handle, ViewportOverrides)>` and
  `restore: RestoreOptions` (which properties restore; default all).
- `layerstate.save {name, viewports?: bool (default true), description?}`.
- `layerstate.export {names, path}` and `layerstate.import {path}` with our format, `.cclayers`:
  `{"format": "cadcraft-layer-states", "version": 1, "states": [{name, description, layers:
  [{name, on, frozen, locked, plot, color, linetype, lineweight, transparency, plotStyle,
  newVpFreeze}], viewports: [...]}]}`. Validate every field; cap sizes; tests with hostile files.
  Import adds missing linetypes from the standard library only (never from the file).

### 2.4 Locked-layer fading

- Renderer: objects on locked layers draw with extra transparency `layer_lock_fade` on screen
  only (not in plots, not in exports).

## 3. UI structure (phases 2–4)

Split `layers.rs` into a module (keep `pub fn dialog` and the public colour helpers where other
files import them):

```
crates/ui-egui/src/layers/
  mod.rs        dialog(), LayerUi state, actions → engine commands
  table.rs      header (sort, column menu), rows (toggles, swatches, previews), multi-select,
                drag source
  filters.rs    sidebar tree, counts, rule editor panel, drop target for groups
  inspector.rs  selected-layer panel, linetype dropdown, viewport "Differs here"
  states.rs     State ▾ menu, save/import/export/rename-delete windows
```

- State struct replaces `LpmState`: `filter: FilterKey` (All, Used, Xref, Overrides, Rule(name),
  Group(name)), `invert`, `search`, `selected: BTreeSet<String>`, `anchor` (for ⇧-click),
  `sort: (Column, desc)`, `scope: Scope { Drawing, Viewport }`, `columns: ColumnSet`,
  `editing_rule: Option<RuleDraft>`, menus open/closed, `highlight_overrides`.
- Sizes and tokens: `spec/layer-manager-spec.md`. Rows 22 pt; custom-painted like the Publish
  window's list (`crates/ui-egui/src/publish.rs` on `feat/publish-dialog` shows the pattern:
  `allocate_exact_size` per row, painter cells, `describe_control` for accessibility).
- Linetype dropdown: reuse `lt_sample` for previews; list loaded linetypes first, then the
  standard library not yet loaded.
- Accessibility: every toggle has an accessible label ("Turn off A-WALL", "Freeze A-WALL in this
  viewport"), checked state through `describe_control`.
- Keyboard: ↑/↓ move, ⇧↑/⇧↓ extend, Space toggles On, F2 renames, Delete deletes (guarded),
  ⌘/Ctrl-A selects all, ⌘/Ctrl-I inverts the filter, Esc closes menus then the window.

## 4. Where the design differs from today on purpose

- One table for both scopes instead of extra "VP …" columns; the switch says what you edit.
- Plot style and New VP Freeze leave the default columns (inspector + column menu).
- Layer states move from toolbar widgets to one State ▾ menu.
- Filter properties are rules in sentences, not a grid of property columns.

## 5. Order of work

See SEQUENCE.md: PR 1 engine (filters, overrides, states, fading), PR 2 window skeleton + table +
inspector, PR 3 filters sidebar + rule editor + groups, PR 4 states menu, settings, references,
polish and docs.

## 6. Tests

- Engine (`crates/engine/tests/layer_filters.rs`, `vplayer_overrides.rs`,
  `layer_states_io.rs`): matching truth tables; rename/delete/merge keep groups right;
  vplayer each override + clear; DXF round-trips; `.cclayers` round-trip and hostile input
  (bad JSON, huge lists, unknown keys, wrong types, path traversal in names).
- UI (`crates/ui-egui/tests/layer_manager.rs`, kittest like `publish_dialog.rs`): opens from
  LAYER; sort; search with wildcards; invert; multi-select toggle = one undo; double-click
  current; viewport scope edits only the viewport; rule editor preview and save; drag onto
  group; linetype dropdown loads a standard linetype; states menu restore; delete guard text;
  compact window at 900×600 @2×. Use `get_all_by_label(..).min_by(y)` or a value predicate when
  a name also appears in the status bar (see `publish_dialog.rs`).
- Visual: `CRAFT_UI_VISUAL_DIR=… cargo test -p cadcraft-ui-egui --test layer_manager`; compare
  with `spec/states/01, 02, 04, 05, 08`.

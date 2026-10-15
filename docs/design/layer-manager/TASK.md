# TASK: Layer Manager

## Goal

Replace the Layer Properties Manager window (`crates/ui-egui/src/layers.rs`, 748 lines at
`fc4e983`) with the Layer Manager in `spec/`: a filter sidebar, one compact layer table, an
inspector for the selected layer, and a **Drawing | Viewport** switch that makes the same
columns edit the active viewport's overrides. Add what the engine lacks for it: rule filters and
groups, more viewport overrides, layer-state import/export and options, and locked-layer fading.

Every change in the window runs an engine command, as today (`layer.*`, `layerstate.*`,
`vplayer`, new `layerfilter.*`), so scripts, the control channel and MCP get the same features.

## Rules (from CLAUDE.md; CLAUDE.md wins on any conflict except the branch rule)

1. **Branch and PRs.** Work on `feat/layer-manager` and open PRs to `NakaSato/cadcraft` `main`;
   do not push to `main` (this overrides CLAUDE.md's "push to main" for this task).
2. **Clean-room.** Never copy AutoCAD's dialog, wording, icons, help text or file formats. No
   `.las` layer-state files: our export format is our own (JSON, below). DXF objects may be read
   and written **only** from Autodesk's published DXF Reference; when the reference doesn't
   describe something, keep it as foreign data (#528 round-trip) instead of guessing.
3. **Assets.** No new image, icon or font files. Every new glyph (eye, eye-off, sun, snowflake,
   open/closed lock, printer, no-printer, new-layer, new-layer-frozen-in-viewports, rule filter,
   group, reference link) is drawn in `crates/ui-egui/src/icons.rs`.
4. **No panics** (`unwrap`, `expect`, indexing that can fail) outside tests.
5. **Strings** through `crate::tl!` / `crate::tf!` (plural via `crate::i18n::count`), with a row
   in `crates/ui-egui/src/i18n/uk.tsv` for each, and every new UI source file listed in
   `crates/ui-egui/src/i18n/tests.rs`. See BRANCHES.md for where to put the rows.
6. **Colours** only from `theme::Tokens` (`accent`, `text`, `text_dim`, `text_faint`, `border`,
   `chrome`, `chrome_dark`, `list_bg`, `list_row`, `control`, `field`, `tab_active`). Layer
   colour swatches show the layer's own colour; that is content, not chrome.
7. **Gate.** `cargo xtask ci` (fmt, clippy -D warnings, tests, assets, layers, wasm) before every
   commit. For UI, render screenshots (`CRAFT_UI_VISUAL_DIR`) and compare with `spec/states/`.
8. **Docs.** When a piece lands, update `docs/ui-parity.md` (row "Layer Properties Manager"),
   `docs/gaps.md`, and `docs/file-format-parity.md` if persistence changes, with their revision
   history lines, as CLAUDE.md's progress-docs rule says.

## Acceptance criteria

1. LAYER (typed, menu, toolbar, Layers palette button) opens the new window. JSON calls to
   `layer.*` never open it.
2. Table: 22 pt rows, columns Current · Name · On · Freeze · Lock · Plot · Color · Linetype ·
   Lineweight · Transparency · Description, plus optional Plot style and Frozen in new viewports
   from the column menu. Click a header to sort (Name, Color, Linetype, Lineweight,
   Transparency); click again to reverse.
3. Click selects; ⇧-click and ⌘/Ctrl-click select several; a toggle clicked on a selected row
   applies to every selected row (one undo step). Double-click makes a layer current. ⌘/Ctrl-A
   selects all shown rows. Rename with F2 or double-click on the name; ⌥N / Alt-N adds a layer.
4. Search box filters by name with `*` and `?` (a plain word means `*word*`), together with the
   chosen filter; "Invert filter" shows the layers the filter leaves out.
5. Filters sidebar: All layers, Used, From references, Viewport overrides (disabled outside a
   layout viewport), then saved rule filters and groups, each with its count.
6. Rule filters: name, Match all / Match any, rules over Name (contains / is / is not / starts
   with), Color, Linetype, Lineweight, State (on, off, frozen, thawed, locked, unlocked,
   plotted, not plotted), Used. The table previews matches while editing. Saved with the
   drawing.
7. Groups: new empty group, group from the layers of the selected objects, drag rows onto a
   group, "Add to Group…" in the inspector, remove from group in the row's context menu. Saved
   with the drawing.
8. Viewport scope (only when a layout viewport is active): Freeze becomes VP freeze; Color,
   Linetype, Lineweight and Transparency edit that viewport's overrides; overridden cells show
   bright text and a 5 pt accent dot (column menu "Mark viewport overrides" turns the marks off);
   the inspector says "Differs here: …" with Remove. The viewport shows and plots them.
9. Inspector: name (rename in place), object count, Make Current or "Current layer", Color
   (palette), Linetype (dropdown: "In this drawing" then "Standard set" with drawn previews;
   picking a standard one loads it), Lineweight, Transparency 0–90, Plot style (named-style
   tables only), Plot, Frozen in new viewports, Description, and Select / Isolate / Merge Into… /
   Add to Group….
10. Layer states menu: restore (one click), Save Current As…, Import…, Export Selected…,
    Rename or Delete…. A state also keeps viewport overrides; restore changes only what the
    state holds. Export/import use our `.cclayers` JSON format.
11. Delete only for unused layers that are not current, 0, Defpoints or from a reference; the
    button explains why when it is disabled.
12. Layers from references (`file|LAYER`) show the file part dimmed; their lock and plot
    can't change; freeze and on/off can (session-only, as today).
13. Locked-layer fading: locked layers draw faded by LAYLOCKFADECTL (0–90, default 50).
14. kittest tests for criteria 2–11 (see IMPLEMENTATION.md §6); engine tests for every new
    command and parameter; DXF round-trip tests for filters, groups and viewport overrides.
15. `cargo xtask ci` passes; screenshots of states 01, 02, 04, 05, 08 attached to each UI PR.

## Open decisions (ask the owner; don't decide alone)

- **Lock fading control.** The owner removed the "Fade locked layers" slider from the window's
  footer on 15 Oct. Options: put it in the gear (Settings) popover, or only in Options ▸ Display.
  Criterion 13 needs the setting either way.
- **Filter persistence format.** Our own XRECORD in a CADCraft named dictionary (simple,
  round-trips in CADCraft only) versus the DXF Reference's layer-filter objects (other programs
  see them). Recommendation: our own, and keep foreign layer filters as round-trip data.
- **Window or palette.** The design is a window; the current one is a window that can dock. Keep
  docking as it is unless the owner says otherwise.

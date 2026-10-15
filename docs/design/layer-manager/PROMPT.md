# Kick-off prompt for Claude Code

Open Claude Code in your CADCraft clone and paste everything below the line.

---

I want to build the new Layer Manager for CADCraft (LAYER, Format ▸ Layers…).

Everything is in `docs/design/layer-manager/` on branch `feat/layer-manager`:

- `README.md`: index
- `TASK.md`: goal, the CLAUDE.md rules that apply, 15 acceptance criteria, open decisions
- `IMPLEMENTATION.md`: code map at `fc4e983`, engine/doc additions, UI module layout, tests
- `BRANCHES.md`: the other open branches, which conflict, and how this work avoids new conflicts
- `SEQUENCE.md`: the steps we go through, one prompt per step
- `spec/layer-manager-spec.md`: sizes, colour tokens, labels, states
- `spec/feature-map.md`: every capability, where it lives in the design, what the engine has
- `spec/states/*.html`: the design in 15 states (open in a browser); PNGs on the `pr-assets`
  branch under `layer-manager/`
- `spec/mockup/Layers.dc.html`: the design-canvas source, reference only

Before writing any code:

1. Read `CLAUDE.md` and follow it, especially the clean-room and asset rules. Where it
   conflicts with these files, `CLAUDE.md` wins, except: stay on `feat/layer-manager` and open
   PRs to `NakaSato/cadcraft` `main` instead of pushing to `main`.
2. `git fetch --all`, check out `feat/layer-manager`, and rebase it on the latest `main` if
   `main` moved. Re-run the conflict check in BRANCHES.md
   (`git merge-tree --write-tree feat/layer-manager <branch>` for each open branch) and tell me
   if anything changed.
3. Read every file in `docs/design/layer-manager/` and look at the state HTML files. Then read
   `crates/ui-egui/src/layers.rs`, `crates/engine/src/cmd/layer.rs` (+ `layer/`),
   `crates/engine/src/cmd/layout.rs` and `layout/vplayer.rs`, `crates/doc/src/tables.rs`
   (`Layer`, `LayerState`), `crates/doc/src/entity.rs` (`Viewport`), `crates/doc/src/library.rs`,
   and how the renderer and the DXF writer use `frozen_layers` / `layer_colors`.
4. Reply with:
   - a short plan for **phase 1 (engine) only**: filters model + commands, viewport overrides
     (including whether `layer_colors` is really not drawn or saved today), layer states with
     viewports and `.cclayers` import/export, LAYLOCKFADECTL, and the tests;
   - anything in IMPLEMENTATION.md that is wrong about the current code;
   - your recommendation on the two open decisions in TASK.md.
   Then stop and wait for me.

Hard rules: never copy AutoCAD's dialog, wording, icons, help text or file formats (no `.las`);
DXF only from the published DXF Reference; no panics; no new asset files (draw glyphs in
`icons.rs`); every string through `tl!`/`tf!` with a `uk.tsv` row placed as BRANCHES.md says;
colours only from `theme::Tokens`; `cargo xtask ci` before every commit; screenshots go to the
`pr-assets` branch, never to `main`.

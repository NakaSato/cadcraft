# Layer Manager: design handoff

A new Layer Manager for CADCraft (LAYER, Format ▸ Layers…). It replaces the current Layer
Properties Manager window in `crates/ui-egui/src/layers.rs` with a three-pane window: filters ·
layer table · inspector, plus a **Drawing | Viewport** switch for viewport overrides.

Branch: `feat/layer-manager` (from `main` at `fc4e983`, Release 0.7.0).

| File | What it is |
|---|---|
| [PROMPT.md](PROMPT.md) | The kick-off prompt to paste into Claude Code |
| [SEQUENCE.md](SEQUENCE.md) | The follow-up prompts, one per step, in four PRs |
| [TASK.md](TASK.md) | Goal, rules from CLAUDE.md, acceptance criteria, open decisions |
| [IMPLEMENTATION.md](IMPLEMENTATION.md) | Code map at `fc4e983`, what to add to doc/engine/ui, persistence, tests |
| [BRANCHES.md](BRANCHES.md) | The other open branches, which ones conflict, and how to avoid new conflicts |
| [spec/layer-manager-spec.md](spec/layer-manager-spec.md) | Every size, colour token, label and state of the design |
| [spec/feature-map.md](spec/feature-map.md) | Every capability → where it is in the design → what the engine has today |
| [spec/mockup/Layers.dc.html](spec/mockup/Layers.dc.html) | The design-canvas source (reference only; logic in its `<script>`) |
| [spec/states/](spec/states/) | 15 states as static HTML (open in a browser) |

PNG renders of the 15 states (2×) are on the `pr-assets` branch under `layer-manager/`, because
CLAUDE.md keeps screenshots out of `main`:
`https://github.com/NakaSato/cadcraft/tree/pr-assets/layer-manager`.

**Before this branch is merged**, decide whether `docs/design/layer-manager/` stays (as the
design record) or is removed in the last PR. Nothing in it is an asset under `cargo xtask assets`
(only `.md` and `.html`).

Clean-room note: the feature list comes from public training material describing what a layer
manager does. The design, wording, icons, sample layer names and file formats are our own. No
Autodesk screenshots, icons, `.las` files or help text are used or committed.

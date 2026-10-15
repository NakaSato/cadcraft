# Open branches and conflicts (checked 15 Oct 2026)

Fork `NakaSato/cadcraft`; `main` = upstream `storytold/cadcraft` `main` = `fc4e983` (Release
0.7.0). Checked with `git merge-tree --write-tree` for every pair.

| Branch | Base | Ahead | Touches |
|---|---|---|---|
| `feat/publish-dialog` | `fc4e983` | 6 | publish.rs, exchange.rs (engine + ui), managers.rs, uk.tsv, i18n tests |
| `feat/publish-lists` | `49ab653` (publish-dialog, one commit before its tip) | 1 | publish.rs, uk.tsv, i18n tests |
| `feat/publish-background` | `8c3780c` (publish-lists) | 2 | publish.rs, uk.tsv, docs/gaps.md, docs/ui-parity.md |
| `feat/menubar-icons` | `e707ab4` (older main) | 358 (includes a merge of upstream) | i18n catalogs (all), menus, icons |
| `feat/raster-images` | `e707ab4` | 3 | apps/*/main.rs, uk.tsv, docs |
| `fix/1014-hatch-boundary-large-drawings` | `e707ab4` | 1 | hatch only |
| `pr-assets` | orphan | – | PR screenshots only; never merged |

## Conflicts between existing branches

| Pair | Files |
|---|---|
| publish-dialog × publish-lists | `crates/ui-egui/src/publish.rs` |
| publish-dialog × publish-background | `crates/ui-egui/src/publish.rs` |
| publish-dialog × raster-images | `apps/cadcraft{,-cli,-web}/src/main.rs` |
| publish-lists × raster-images | `apps/cadcraft{,-cli,-web}/src/main.rs` |
| publish-background × raster-images | `apps/*/main.rs`, `docs/gaps.md`, `docs/ui-parity.md` |
| menubar-icons × publish-background | `docs/gaps.md`, `docs/ui-parity.md` |
| menubar-icons × raster-images | `docs/file-format-parity.md`, `docs/gaps.md`, `docs/ui-parity.md` |

All other pairs merge cleanly.

**Why publish.rs conflicts:** publish-lists and publish-background were branched from
`49ab653`, before `e81ce09` ("Publish Sheets edits every page setting for a run") landed on
publish-dialog. Fix: rebase publish-lists onto publish-dialog, then publish-background onto
publish-lists, resolving `sheet_tab` / `SetupChange` once.

## `feat/layer-manager` against them

The branch is created from `main` (`fc4e983`) and its first commit adds only
`docs/design/layer-manager/`, so it conflicts with nothing today. Later work will touch files
other branches touch too; keep conflicts small:

- **`uk.tsv`**: every branch appends at the end, so two branches always conflict there.
  Put layer rows in their own block with a comment header `# Layer Manager`, placed **right
  after the existing `Layer …` rows (around line 453–460 at `fc4e983`)**, not at the end of the
  file. Never reorder or rewrite other rows.
- **`i18n/tests.rs`**: add the new `layers/*.rs` entries next to the existing `layers.rs` line,
  not at the end of the list.
- **`icons.rs`**: menubar-icons adds many icons. Add the layer glyphs in one contiguous block and
  new `Icon` variants at one place; expect a small conflict with menubar-icons and resolve it by
  keeping both.
- **`docs/gaps.md`, `docs/ui-parity.md`**: change only the Layer rows; put revision-history lines
  in a separate commit at the end of each PR so they are easy to re-apply after a rebase.
- **`managers.rs`, `publish.rs`, `apps/*/main.rs`**: don't touch them in this work.
- Rebase `feat/layer-manager` on `main` before each PR; if a publish or menubar branch merged
  first, rebase onto the new `main` and re-run `cargo xtask ci`.

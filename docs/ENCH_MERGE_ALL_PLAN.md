# PhotoCraft Ench: merge-all implementation brief

Target branch: `merge-all` in `Bntagaaa/photocraft-forked`.

## Read before coding
- `AGENTS.md`, `docs/roadmap.md`, `docs/scorecard.md`, `docs/development.md`.
- Original GitHub issues: [#214](https://github.com/storytold/photocraft/issues/214), [#1079](https://github.com/storytold/photocraft/issues/1079), [#1116](https://github.com/storytold/photocraft/issues/1116).
- The upstream repository is `storytold/photocraft`. Always check current code; do not cherry-pick features already integrated into `main`.

## Objective
Implement the FOUR remaining PhotoCraft Ench usability improvements, each in its own commit on `merge-all`. Preserve compatibility with upstream, use existing command/session mechanisms, and do not modify `main` or open/merge upstream PRs.

### 1. Interactive rounded rectangle corner handles
Files to inspect: `crates/ui-egui/src/vector_ui.rs`, canvas drawing/pointer handling, `crates/doc/src/vector.rs`, shape commands.
Existing `LiveShape::Rect { rect, radii: [f64; 4] }` already models per-corner radii. Existing Properties UI has independent corner values and a uniform value. Implement four draggable in-canvas radius handles for a selected editable live rectangle. Default drag adjusts all linked corners; Alt/Option-drag changes only the selected corner (per upstream #214). Ensure correct coordinate transformations for zoom/pan, rotated/transformed shapes, hit testing versus existing transform/path handles, normalized/clamped radii, immediate live preview, Properties sync, save/reopen and exactly one undo entry per drag. Keep numeric controls. Do not rasterize or corrupt PSD/.pcraft.
Add unit and UI/control-channel tests for single and linked edits, modifier behaviour, edge conditions, undo/redo.

### 2. Consistent toolbar across themes
Inspect `crates/ui-egui/src/theme.rs`, `panels.rs`, options bars, UI state, preferences. Reproduce user-reported differences among Pro, ProMedium, Studio, StudioLight and Classic. Preserve intentional layout differences, but ensure all equivalent core actions are reachable, using overflow/adaptive layout where necessary. Do not simply copy Pro look to Studio. Verify narrow windows, HiDPI and no dead controls. See #1116; fix only confirmed gaps; avoid unrelated theme redesign.

### 3. Convert multiple selected layers into ONE Smart Object
Inspect `crates/engine/src/smart_cmds.rs` (current `convert` targets `layer_param` only), selection storage in `Session::DocState::selected_layers`, `crates/doc`, nested document encoding, smart source serialization, `crates/io`, PSD exporter.
Convert all selected layers into a single parent Smart Object, retaining original editable layered contents and their relative order, composited appearance, transforms, transparency, masks, styles and blending as closely as the current model supports. Support selected noncontiguous layers and mixed types correctly or return an explicit unsupported error without modifying document. Handle locks, groups, and invalid targets gracefully. Undo restores all source layers and selection. Save/reopen native .pcraft retains nested contents; do not falsely claim PSD fidelity if export still flattens smart objects. Add regression tests directly reproducing upstream #1079. Ensure single-layer conversion remains unchanged.

### 4. Transform controls & confirmation placement
Inspect `crates/ui-egui/src/transform_tool.rs`, options bar, menus, `crates/engine/src/transform_cmds.rs`. Surface existing Scale, Rotate, Skew, Distort, Perspective, Warp, Flip Horizontal/Vertical, Rotate 90/180 actions in the appropriate context menu when supported. Move Confirm/Cancel near the active transform parameters on the toolbar, with responsive/overflow positioning, while preserving Escape/Enter shortcuts. Do not duplicate operations already present or regress active-transform history / previews. Cover themes, narrow windows and undo/redo.

## Work protocol
1. Inspect repo status and HEAD; never force-push.
2. Implement changes in order: rounded corners; toolbar; transform; smart objects.
3. Make one self-contained commit per feature and keep `merge-all` buildable.
4. For each area run crate tests, `cargo fmt --check`, `cargo clippy`, `cargo xtask layers`; finish with `cargo xtask ci`, plus `cargo xtask test-corpus` when IO/PSD/compose is touched. Follow `AGENTS.md` for all required gates.
5. Use desktop control channel and `ui.screenshot` for visual verification instead of OS-level synthetic input. Compare default and alternate themes, 100%/HiDPI, narrow/wide windows.
6. Never report a feature as complete unless its tests/build succeed; report remaining known gaps candidly.

## Distribution
Create a test build of `merge-all` for Linux x86_64, preferably AppImage and .deb via supported existing packaging/workflows. Do not publish a stable release or change `main`. Provide artifact/run links and exact commits; if CI runners or required dependencies are missing, say which step blocked.

## Acceptance report
Provide a matrix of all four features with status (done/partial/blocked), changed files, commit hashes, test/build results, visual screenshots, and unresolved limitations.

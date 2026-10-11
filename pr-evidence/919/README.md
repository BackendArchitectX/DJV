# PR #919 — current-head Windows GUI verification

Feature PR: https://github.com/grizzlypeak3d/DJV/pull/919  
Exact tested source commit: `b6b70d3fe1feb15cf2f9ebc9fa282979b8d68634`  
Evidence branch: `pr-919-evidence` (screenshots kept separate from feature code and CI)

## Results

- Windows Debug build passed at the exact commit.
- CTest: **1/1 top-level CTest targets passed; 17 internal test classes; 0 failures**, including Information search baseline and implied-AND classes.
- Exact-head fork push CI: 12 successful jobs; Linux/GLES3 CTest: 12/12 each. Upstream pull-request workflow still `action_required` and has **not run**.
- GUI cases passed: cross-section filtering; reversed terms; absent terms; both Duration fields; filtered and empty Copy; MP4/JPEG/EXIF media switching.
- Real EXIF data: `Exif:FocalLength = 50` and `Exif:PhotographicSensitivity = 200`; filtered Copy includes only these rows.
- Four selectable `Cubes.0001.exr` video layers: Combined, DiffCol, GlossDir, Normal.
- During EXR → JPEG → EXR with `type` query: `RGB F16 → RGB U8 → RGB F16`, with no persistent stale result or crash.
- Close All clears Information. A seeded PowerShell clipboard sentinel confirmed empty Copy **replaces** stale clipboard content.

## Current-head screenshots

### Real EXIF query

![Real EXIF search](./11-exif-search-exact-head.webp)

### Filtered EXIF Copy

![EXIF Copy output](./12-exif-copy-output.webp)

### Four-layer EXR menu

![Multilayer EXR](./13-exr-four-layers.webp)

### Pixel-type round trip

![EXR JPEG EXR type roundtrip](./14-pixel-type-exr-jpeg-exr.webp)

### Close All

![Close All clears Information](./15-close-all-clears-information.webp)

### Clipboard sentinel

![Clipboard sentinel passed](./16-clipboard-sentinel-pass.webp)

## Limits and pending decisions

- These are real manual GUI screenshots from a clean exact-head Windows build, **not** automated Information-panel screenshot tests.
- No persistent stale information or crash appeared in the tested states. A **transient** observer callback may still refresh before the new player's video-layer index is initialized. That timing has **not been directly instrumented or ruled out**.
- Cross-row implied-AND semantics, grouping of unquoted multiword labels, and any explicit `AND` syntax remain pending maintainer confirmation.
- Upstream PR workflow approval remains pending; the successful fork CI does not replace it.

Original PR description retains ten earlier screenshots separately as historical evidence.

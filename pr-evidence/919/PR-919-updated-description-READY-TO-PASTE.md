## Summary

Implements **implied-AND search** in DJV's **Information** panel for issue #316 and strengthens the **pull-request test workflow** using the focused workflow correction from PR #918, now transferred as commit `b6b70d3fe1feb15cf2f9ebc9fa282979b8d68634`. These code changes are consolidated into one reviewable PR; the duplicated Recent Files fixture correction is included only once.

Information search is case-insensitive across field labels and values in **File, Video, Audio, and Metadata**. The provisional behavior requires every *parsed* query term to match somewhere in the current Information panel; if all terms exist, rows matching at least one term are shown. If any required term is missing, no rows are shown. This cross-row behavior and the handling of unquoted multiword field names are **pending maintainer confirmation**. The implementation is ready for maintainer review; the exact search semantics and upstream workflow authorization remain open review gates.

The pull-request workflow correction changes the Linux build to use `ci-linux`, which enables `DJV_TESTS`, removes `--rerun-failed` from the ordinary PR test command, adds `--no-tests=error`, and imposes a 600-second timeout. This addresses the previously observed false-positive scenario in which the PR workflow completed with **"No tests were found!!!"**.

## Original #919 feature summary (verbatim)

> Provisional **implied-AND filtering** for the DJV **Information** panel (related to #316). Search is case-insensitive across field labels and values in File, Video, Audio and Metadata. When every **parsed** term is found somewhere in the current panel, display matching rows; otherwise display none. The exact semantics are pending maintainer feedback, so maintainer feedback is requested before finalizing the search contract.

## Original #918 summary and rationale (preserved)

The independent #918 PR described the following scope before consolidation:

> Fix the PR workflow to ensure tests are executed and prevent false-positive CI results when no tests are discovered.

Its implementation and motivation were:

- Use the existing `ci-linux` configuration to enable `DJV_TESTS`.
- Remove `--rerun-failed` to execute the complete registered test suite.
- Add `--no-tests=error` to fail when no tests are discovered.
- Set a 600-second timeout for test execution.
- A previous successful PR workflow reported **`No tests were found!!!`**, allowing a green test step without executing tests.

Its original validation stated that the workflow change was limited to two lines and that `git diff --check` passed, with Linux CI pending at the time of submission. Subsequently, its separate fork branch [CI run #38093051943](https://github.com/BackendArchitectX/DJV/actions/runs/38093051943) succeeded. This historical run validates **that branch only**, not the new combined commit.

## Changes

### Information search (#316; original PR #919)

- Preserve baseline case-insensitive substring matching against field labels and values.
- Apply provisional panel-wide implied-AND semantics across parsed query terms.
- Show matching rows in the existing section order while suppressing unmatched rows.
- Support recognized multiword field labels and the `filename`/`dimensions` aliases for existing `Name`/`Resolution` labels.
- Update filtering when the selected media/reference and video layer change.
- Make **Copy** use the displayed filtered rows, without headings for empty sections.
- Register `InfoSearchBaselineTest` and `InfoSearchAndTest` in the existing `djv-test` executable.

### Pull-request test discovery (original PR #918)

- Build with `sh djv/etc/Linux/sbuild.sh djv Debug ci-linux` so CMake enables `DJV_TESTS`.
- Run `xvfb-run ctest --verbose --output-on-failure --no-tests=error --timeout 600 -C Debug`.
- Prevent a successful PR check when CTest finds zero tests.
- Keep this change in `.github/workflows/pr-check.yml`; it does not independently change all push-workflow test commands.

### Capture fixture (shared change, applied once)

- In `tests/capture/capture.json`, replace the unsupported `Charlie.jpg` Recent Files test input with `ColorPatternPortrait.png`, adjusting the query from `charl` to `portrait`.
- This fixes an existing GLES3 capture scenario; **it is not an automated Information-panel UI test**.

## Scope and behavior requiring maintainer guidance

Issue #316's discussion favors **implied AND per word**, rather than a mandatory explicit `AND` operator. The current implementation also groups a small fixed set of known field labels as phrases (for example, `Pixel Aspect Ratio`), regardless of token order. These interpretations can differ if individual query words appear across separate rows.

Please confirm:

1. For `name duration resolution`, should all three terms first have to exist somewhere in the current panel, followed by displaying the union of matching rows, or should available matches remain visible when one requested term is absent?
2. Should unquoted words always remain independent terms, or should recognized multiword field names receive special treatment? In particular, what should happen when `pixel`, `aspect`, and `ratio` occur in separate fields?
3. Is explicit `AND` syntax desired in addition to the maintainer-requested implied AND? This is not implemented in the provisional parser.

Regular expressions and saved searches remain **out of scope**.

## Tests and verification — evidence boundaries

**New current head after consolidation:** [`b6b70d3fe1feb15cf2f9ebc9fa282979b8d68634`](https://github.com/BackendArchitectX/DJV/commit/b6b70d3fe1feb15cf2f9ebc9fa282979b8d68634), a child of the earlier successful feature head `6f814940a473`. Its single commit changes only `.github/workflows/pr-check.yml` (two lines added, two removed). The combined PR has **9 changed files and 6 commits**, with the shared PNG fixture correction included once.

- [Consolidated-head fork CI run #38123431720](https://github.com/BackendArchitectX/DJV/actions/runs/38123431720) **completed successfully** on commit `b6b70d3fe1feb15cf2f9ebc9fa282979b8d68634`: **12 successful matrix jobs, 1 release job skipped (expected for a non-tag branch)**. Linux, Linux GLES3, Windows, macOS and packaging jobs succeeded.
- [Consolidated-head upstream PR run #38123434957](https://github.com/grizzlypeak3d/DJV/actions/runs/38123434957) remains **`action_required`** with no upstream jobs executed. An authorized upstream maintainer/repository action is needed; the successful fork push workflow **does not** constitute a passing upstream PR workflow.

| Validation | Evidence | Limitation |
| --- | --- | --- |
| **Consolidated-head fork CI** | [Run #38123431720](https://github.com/BackendArchitectX/DJV/actions/runs/38123431720), commit `b6b70d3`: **12 successful matrix jobs; release skipped** | Fork **push** workflow, not upstream **pull_request** workflow |
| **Exact-head Linux CTest** | `linux-build`: **12/12 CTest targets passed**, `djv-test` ran **17 internal classes, 0 failures**; both `InfoSearchBaselineTest` and `InfoSearchAndTest` executed | Confirms registered automated tests, not complete manual GUI behavior |
| **Exact-head Linux GLES3 CTest** | `linux-gles3-build`: **12/12 CTest targets passed**, including `djv-capture-recent-files` and both Information search test classes | Recent Files capture is not an Information search GUI integration test |
| **Exact-head Windows Debug build** | **Passed** on clean `b6b70d3`, with rebuilt `djvUI`, `djv-test` and `djv` | Manual Windows verification, not upstream PR workflow |
| **Exact-head Windows Debug CTest** | **1/1 CTest target passed**, with **17 internal test classes**, zero failures; both Information search test classes executed | `DJV_TESTS_NO_GL=ON`: no Windows automated GUI capture; 17 classes are not 17 CTest targets |
| Search unit coverage | Cases for empty/whitespace queries, casing, multirow matches, `filename`/`dimensions`, duplicates, recognized phrases, term-order invariance, missing terms, metadata, changed input and optional `Source Channels` | Tests reflect provisional semantics, not yet maintainer-approved behavior |
| Feature-branch fork CI (pre-consolidation) | [Run 38103329151, attempt 2](https://github.com/BackendArchitectX/DJV/actions/runs/38103329151/attempts/2), head `6f814940a47351a80f814f48a5c7d6a45faea44b`: **12 successful build/test jobs, release skipped**, after a single successful infrastructure-only retry | This is the **pre-consolidation SHA**, not proof that the new combined head passed |
| CI-fix fork branch | [Run 38093051943](https://github.com/BackendArchitectX/DJV/actions/runs/38093051943), head `2eb310c184eb2d309ea625e36b103b9f99ff5885`: **success** | Different branch and head from #919 |
| Upstream PR workflow | [Run 38123434957](https://github.com/grizzlypeak3d/DJV/actions/runs/38123434957): **`action_required`**, zero jobs started | Requires upstream repository/maintainer action; no upstream CI success claim |
| Actual Information UI | Ten preserved historical screenshots below **plus six new clean-head manual GUI screenshots** linked in the new evidence section | Historical `-dirty` captures remain labeled; new screenshots show clean `b6b70d3` |
| **Real JPEG EXIF** | **Passed at clean head:** `focallength photographic` shows `Exif:FocalLength=50` and `Exif:PhotographicSensitivity=200`; Copy preserves both | This proves the two observed metadata fields, not all possible EXIF variants |

**The workflow-fix commit is included in #919**, and the new exact-head fork CI has now passed. The Linux and GLES3 logs confirm positive test discovery and execution, including both Information search test classes and the Recent Files capture regression. The change to `.github/workflows/pr-check.yml` specifically affects the upstream `pull_request` workflow, which is still blocked at `action_required`; the successful fork **push** CI does not validate execution of that separate upstream workflow. Upstream approval remains a distinct requirement.

### Consolidated-head GitHub Actions verification

- [Fork push workflow #38123431720](https://github.com/BackendArchitectX/DJV/actions/runs/38123431720) completed **successfully** on `b6b70d3`: **12/12 matrix jobs passed**; the separate release job was **skipped as expected**.
- [Linux build job](https://github.com/BackendArchitectX/DJV/actions/runs/38123431720/job/114422713147): **12/12 CTest targets passed**; the internal `djv-test` executable ran **17 test classes**, **0 failures**, including both Information search classes.
- [Linux GLES3 build job](https://github.com/BackendArchitectX/DJV/actions/runs/38123431720/job/114422713387): **12/12 CTest targets passed**, including `djv-capture-recent-files`; the same **17 internal classes** passed.
- **Windows build, Windows Python build, macOS builds, Linux system build and packaging jobs** also completed successfully within the fork workflow.
- [Upstream PR workflow #38123434957](https://github.com/grizzlypeak3d/DJV/actions/runs/38123434957) remains **`action_required`**; no upstream PR tests have executed. The fork CI result must not be relabeled as upstream approval.

### Local Windows Debug commands previously used

```bat
cmake --build R:/DJV-316-build/build-Debug --config Debug --parallel 2
ctest --test-dir R:/DJV-316-build/build-Debug --verbose --output-on-failure --no-tests=error --timeout 600 -C Debug
```

Previously observed test excerpts:

```text
Running test: tests::InfoSearchBaselineTest
Running test: tests::InfoSearchAndTest
Tests run: 17
Failures: 0
1/1 Test #1: djv-test ......................... Passed
100% tests passed out of 1
CTEST_EXIT_CODE=0
```

These observations relate to the pre-consolidation local build and must not be relabeled as a run of the new head.

## Manual test media — MP4, two JPEGs and four-layer EXR

| Asset | Verified observation | Limitation |
| --- | --- | --- |
| `sample-5s-360p.mp4` | Information search across File/Video/Audio; two Duration rows; multiword label; Copy; switch away and back | Source file is a video asset, not a screen recording |
| `sample-photo-1920x1080.jpg` | Opened in DJV; 1920×1080 resolution, JPEG-related metadata, empty Audio section; `name channels` gives no rows under provisional strict AND | Development-session screenshot only |
| `sample-with-exif-400x300.jpg` | **Passed** on clean head: real EXIF keys and values, `focallength photographic` filter and Copy; MP4/JPEG/EXIF switching | Earlier inventory screenshot remains historical only |
| `etc/SampleData/Cubes.0001.exr` | **Passed targeted GUI checks**: 4 layers (Combined, DiffCol, GlossDir, Normal), `resolution` search, and `type` query EXR → JPEG → EXR (RGB F16 → RGB U8 → RGB F16) | Final states correct; transient observer callback timing uninstrumented |

## Preserved actual UI screenshots

These are the ten screenshot links already documented in original PR #919. Their **observed results and evidence boundaries are deliberately retained**. Screenshots captured with a `-dirty` title bar do not prove behavior from a clean, post-commit head.

### 1. Cross-section search — `name channels`

Observed File → Name and Audio → Channels for the video.

![01 cross section name channels](https://raw.githubusercontent.com/BackendArchitectX/DJV/e54891f1ea0a067c4aaa2d7ada5b8ff91ee9ec78/pr-evidence/919/01-cross-section-name-channels.png)

### 2. Duplicate label — `duration`

Observed both Video → Duration and Audio → Duration.

![02 both video and audio durations](https://raw.githubusercontent.com/BackendArchitectX/DJV/e54891f1ea0a067c4aaa2d7ada5b8ff91ee9ec78/pr-evidence/919/02-both-video-and-audio-durations.png)

### 3. Multiword label — `pixel aspect ratio channels`

Observed Video → Pixel Aspect Ratio and Audio → Channels using current recognized-label parsing.

![03 multiword pixel aspect ratio channels](https://raw.githubusercontent.com/BackendArchitectX/DJV/e54891f1ea0a067c4aaa2d7ada5b8ff91ee9ec78/pr-evidence/919/03-multiword-pixel-aspect-ratio-channels.png)

### 4. Switch to JPEG without audio

With `name channels` active, observed no matching rows under provisional strict panel-wide AND.

![04 media switch no audio no results](https://raw.githubusercontent.com/BackendArchitectX/DJV/e54891f1ea0a067c4aaa2d7ada5b8ff91ee9ec78/pr-evidence/919/04-media-switch-no-audio-no-results.png)

### 5. Return to MP4

Previously matching rows reappeared.

![05 media switch return to video](https://raw.githubusercontent.com/BackendArchitectX/DJV/e54891f1ea0a067c4aaa2d7ada5b8ff91ee9ec78/pr-evidence/919/05-media-switch-return-to-video.png)

### 6. Copy filtered results

Observed clipboard text containing only nonempty File/Audio sections, without empty Video/Metadata headings.

![06 filtered clipboard only visible sections](https://raw.githubusercontent.com/BackendArchitectX/DJV/e54891f1ea0a067c4aaa2d7ada5b8ff91ee9ec78/pr-evidence/919/06-filtered-clipboard-only-visible-sections.png)

### 7. JPEG media details

Observed `sample-photo-1920x1080.jpg` with resolution `1920 1080`, JPEG-related metadata, and empty Audio section.

![07 image jpeg metadata 1920x1080](https://raw.githubusercontent.com/BackendArchitectX/DJV/e54891f1ea0a067c4aaa2d7ada5b8ff91ee9ec78/pr-evidence/919/07-image-jpeg-metadata-1920x1080.png)

### 8. Two-JPEG Downloads inventory (cropped)

Shows file names only; not proof that the EXIF sample was opened or successfully searched in DJV. The screenshot was cropped to hide unrelated local files.

![08 downloads two jpeg files CROPPED](https://raw.githubusercontent.com/BackendArchitectX/DJV/e54891f1ea0a067c4aaa2d7ada5b8ff91ee9ec78/pr-evidence/919/08-downloads-two-jpeg-files-CROPPED.png)

### 9. Local Windows Debug smoke check — `name channels`

Observed File → Name and Audio → Channels; title bar displayed `8045921-dirty`. This does not independently prove a clean-head replay, reverse-order query, optional Source Channels or upstream CI.

![09 local GUI name channels working tree](https://raw.githubusercontent.com/BackendArchitectX/DJV/8a41a7fce578ac1a790ad0f403a8d2296c812d93/pr-evidence/919/09-local-gui-name-channels-working-tree.png)

### 10. Reverse-order Windows Debug smoke check — `channels name`

Observed the same rows as `name channels`, File → Name (`sample-5s-360p.mp4`) and Audio → Channels (`2`); title bar displayed `8045921-dirty`. Does not establish all permutations or clean-head behavior.

![10 reverse-order channels name local GUI](https://raw.githubusercontent.com/BackendArchitectX/DJV/436366fd4e6818186d16fa1bff50e68cbd72fc3f/pr-evidence/919/10-local-gui-channels-name-working-tree.png)

## Clean-head Windows GUI and clipboard screenshots (new)

The six new images were captured from the actual Windows Debug application at `b6b70d3`, with irrelevant areas cropped. They are hosted in the existing `pr-919-evidence` branch at immutable [evidence commit `85ed0e6fa`](https://github.com/BackendArchitectX/DJV/commit/85ed0e6fa90cf6e35ef454c0c9efe0369a15e707), **not** in the feature branch. The earlier 10 screenshots remain above as historical evidence. A [separate evidence gallery](https://github.com/BackendArchitectX/DJV/blob/pr-919-evidence/pr-evidence/919/README.md) provides a compact summary.

### 11. Real EXIF search
Query `focallength photographic` shows exactly two metadata rows: `Exif:FocalLength: 50` and `Exif:PhotographicSensitivity: 200`.

![11 EXIF search current head](https://raw.githubusercontent.com/BackendArchitectX/DJV/85ed0e6fa90cf6e35ef454c0c9efe0369a15e707/pr-evidence/919/11-exif-search-exact-head.webp)

### 12. EXIF filtered Copy
Copied text preserves both EXIF rows under Metadata, excluding unrelated fields.

![12 EXIF Copy](https://raw.githubusercontent.com/BackendArchitectX/DJV/85ed0e6fa90cf6e35ef454c0c9efe0369a15e707/pr-evidence/919/12-exif-copy-output.webp)

### 13. Four-layer EXR
`Cubes.0001.exr` exposes Combined, DiffCol, GlossDir and Normal layers via File → Layers.

![13 EXR layers](https://raw.githubusercontent.com/BackendArchitectX/DJV/85ed0e6fa90cf6e35ef454c0c9efe0369a15e707/pr-evidence/919/13-exr-four-layers.webp)

### 14. Pixel type refresh across media
While `type` stays active, EXR → JPEG → EXR produces **RGB F16 → RGB U8 → RGB F16** with no persistent stale result or observed crash.

![14 EXR JPEG EXR type roundtrip](https://raw.githubusercontent.com/BackendArchitectX/DJV/85ed0e6fa90cf6e35ef454c0c9efe0369a15e707/pr-evidence/919/14-pixel-type-exr-jpeg-exr.webp)

### 15. Close All removes old information
With no media open, all Information sections contain zero result rows.

![15 Close All clears rows](https://raw.githubusercontent.com/BackendArchitectX/DJV/85ed0e6fa90cf6e35ef454c0c9efe0369a15e707/pr-evidence/919/15-close-all-clears-information.webp)

### 16. Clipboard clearing verified with a sentinel
A PowerShell test preloaded `DJV_PR919_OLD_CONTENT`, invoked DJV's Copy with an empty Information panel, then verified the old clipboard value was replaced: **PASS: Clipboard was cleared**.

![16 Clipboard sentinel PASS](https://raw.githubusercontent.com/BackendArchitectX/DJV/85ed0e6fa90cf6e35ef454c0c9efe0369a15e707/pr-evidence/919/16-clipboard-sentinel-pass.webp)

**Evidence limitations:** These are real manual GUI checks, not automated Information-panel GUI-capture tests. They prove correct final content, not absence of a transient intermediate observer refresh; that potential stale-layer initialization path has not been directly instrumented. Windows local CTest uses `DJV_TESTS_NO_GL=ON`.

## Reproduction steps

1. Open `sample-5s-360p.mp4`; choose Tools → Information and search `name channels`. Confirm File → Name and Audio → Channels.
2. Search `duration`. Confirm separate Video and Audio Duration rows.
3. Search `pixel aspect ratio channels`. Confirm the current recognized-label behavior.
4. While searching `name channels`, switch to `sample-photo-1920x1080.jpg`. Confirm that Audio Channels is absent and the strict filter returns no rows.
5. Switch back to the MP4. Confirm the expected filtered rows reappear.
6. Use Copy and inspect clipboard text. Confirm only matching nonempty sections are included.
7. **Passed at clean head:** EXIF search `focallength photographic` displays FocalLength 50 and PhotographicSensitivity 200; filtered Copy and MP4/JPEG/EXIF switching were manually verified (#11–12).
8. **Passed targeted GUI switching:** four-layer EXR ↔ single-layer JPEG with active `resolution` and `type` queries; switched layers and used Close All without persistent stale data or crash (#13–15). **Remaining limitation:** intermediate observer callback timing was not instrumented.
9. **Passed current-head Windows Debug:** build and CTest 1/1 with 17 internal classes, zero failures, both search test classes run. Linux/GLES3 fork CI separately passed 12/12 targets; upstream PR CI remains unexecuted.


## Additional engineering checks worth completing before review

This historical risk inventory contains both **completed** manual regression checks and **unresolved** maintainer or instrumentation questions. The unit tests cover related parser scenarios; do not treat them as approval of provisional semantics.

| Priority | Scenario | Exact acceptance test |
| --- | --- | --- |
| P0 | Implied-AND semantics | With `Pixel Type`, `Aspect`, and `Ratio` as separate rows, `pixel aspect ratio` must follow the maintainer-agreed word/phrase semantics; verify both union-selection and missing-term behavior. |
| P0 | Upstream test-discovery check | `pr-check.yml` must run on the **combined** head, discover at least one CTest target, execute the registered tests, report zero failures and fail if no tests exist. Pending maintainer approval. |
| P0 | Exact-head CI | **Verified:** 12 successful matrix jobs, one skipped release job; Linux and GLES3 CTest each passed 12/12. Preserve exact-head links and distinguish fork CI from pending upstream PR CI. |
| P1 | Input properties | Validate token-order invariance, idempotence, case normalization, whitespace, missing terms, multiword labels, and alias behavior; add randomized permutations only after semantics are fixed. |
| P1 | Media lifecycle | **Targeted GUI checks passed:** four-layer EXR ↔ JPEG, layer selection, `resolution`, `type`, Close All. No persistent stale rows or crash; **transient initialization timing not instrumented**. |
| P1 | Clipboard consistency | **Passed observed GUI checks:** filtered File/Audio rows, duplicate Duration rows, EXIF Copy and empty Copy; a seeded PowerShell sentinel confirms no stale clipboard text. Multiline-value GUI coverage remains limited. |
| P1 | Real EXIF evidence | **Passed clean-head GUI:** real FocalLength/PhotographicSensitivity metadata, `focallength photographic` search, filtered Copy, MP4/JPEG/EXIF switching. See screenshots #11–12. |
| P1 | GUI capture | Add an Information-panel-specific capture scenario if the harness can reliably enter queries and assert visible rows; do not mistake Recent Files capture coverage for search UI coverage. |
| P2 | Unicode and punctuation | Define case/normalization expectations for arbitrary metadata, quoted text, `AND` literals, embedded punctuation and non-ASCII strings. Avoid promising unsupported parser syntax. |
| P2 | Performance | Measure typical and worst-case metadata size and query length before adding caches, indexing, regex or search infrastructure. |

**Implementation caution:** `InfoWidget::setPlayer()` attaches the media-reference observer (which can call `_widgetUpdate()`) before the video-layer observer; the old `videoLayer` is not explicitly reset for a non-null player. Validate observer initialization order and media-layer bounds before labeling this a confirmed bug.

**Review scope caution:** The CI fix is independently useful to upstream. Its inclusion in this consolidated feature PR may delay that improvement; the workflow correction is therefore preserved in a separate, focused commit within #919. PR #918 is already closed as superseded. Maintainers can request a change in scope if they prefer independent review, but no additional PR is proposed here.

## Outstanding work / review gates

- [x] Transfer **only** PR #918's unique workflow change from commit `bf5f586a6fe074cddd65164b45b24e10cc753678`, now applied as [`b6b70d3`](https://github.com/BackendArchitectX/DJV/commit/b6b70d3fe1feb15cf2f9ebc9fa282979b8d68634), without its duplicate capture-fixture commit.
- [x] Verify the combined diff contains the PNG fixture correction once and includes `.github/workflows/pr-check.yml` (9 changed files, 6 commits).
- [x] Verify exact-head fork CI for commit `b6b70d3`: 12 successful jobs, expected skipped release; Linux and GLES3 CTest each passed 12/12 with both Information search test classes executing.
- [ ] Obtain upstream permission/approval for the pending PR workflow; do not claim upstream jobs ran while it remains `action_required`.
- [ ] Obtain maintainer guidance on strict panel-wide AND and unquoted multiword label parsing.
- [ ] Reproduce or rule out the potential stale video-layer initialization path during media switching.
- [x] Perform clean-head GUI and EXIF checks, including filtered/empty Copy, four-layer EXR/JPEG switching and clipboard sentinel; preserve evidence limitations.
- [x] Publish the unified PR description in #919, preserving both original summaries, their validation limitations and all **10 embedded screenshot links**.
- [x] Close PR #918 as superseded, without merging it or deleting its branch; keep its discussion and commits available for auditability.

## Consolidation provenance

- Primary feature: [issue #316](https://github.com/grizzlypeak3d/DJV/issues/316), feature PR [#919](https://github.com/grizzlypeak3d/DJV/pull/919).
- Independently proposed CI fix: [PR #918](https://github.com/grizzlypeak3d/DJV/pull/918), originating workflow commit [`bf5f586a`](https://github.com/BackendArchitectX/DJV/commit/bf5f586a6fe074cddd65164b45b24e10cc753678); transferred with identical two-line diff in feature commit [`b6b70d3`](https://github.com/BackendArchitectX/DJV/commit/b6b70d3fe1feb15cf2f9ebc9fa282979b8d68634).
- The shared capture-fixture correction is already present in the feature branch; the former CI PR's separate fixture commit must not be replayed.

This PR consolidates the Information search implementation and CI test-discovery correction previously proposed in #918, which has now been closed as superseded.

The implementation is ready for maintainer review. Exact-head fork CI and manual Windows Debug/EXIF/clipboard verification have **passed**. The remaining gates are **upstream PR workflow approval**, **maintainer confirmation of implied-AND and multiword-label semantics**, and **the uninstrumented potential transient video-layer initialization refresh**. No additional PR or CI reruns are proposed without an actionable finding.
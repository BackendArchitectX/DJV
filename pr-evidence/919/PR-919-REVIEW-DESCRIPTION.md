## Summary

Addresses [#316](https://github.com/grizzlypeak3d/DJV/issues/316) with **provisional implied-AND search** in DJV's Information panel, and consolidates the focused test-discovery correction from superseded [PR #918](https://github.com/grizzlypeak3d/DJV/pull/918). The shared Recent Files capture fixture correction is included only once.

**Search semantics are pending maintainer confirmation.** Currently, every *parsed* term must match somewhere in the active panel; if all terms exist, rows matching **at least one** term are displayed in File, Video, Audio, Metadata section order. Otherwise no rows are displayed. Matching is case-insensitive across field labels and values. The parser recognizes a few multiword field labels and the Filename/Dimensions aliases for Name/Resolution. Literal `AND`, quoted phrases, regex and saved searches are not implemented.

## Changes

- Extract Information row matching into `InfoSearch.h` and register `InfoSearchBaselineTest` and `InfoSearchAndTest` within `djv-test`.
- Update results when selected media/reference or video layer changes; Copy exports currently displayed rows without empty section headings.
- Apply PR #918's isolated `.github/workflows/pr-check.yml` fix in commit [`b6b70d3`](https://github.com/BackendArchitectX/DJV/commit/b6b70d3fe1feb15cf2f9ebc9fa282979b8d68634): build with `ci-linux` (enables `DJV_TESTS`), remove `--rerun-failed`, add `--no-tests=error` and a 600-second timeout. Broader push workflows remain unchanged.
- Correct `tests/capture/capture.json` Recent Files fixture by substituting `ColorPatternPortrait.png` for unavailable `Charlie.jpg`, and `portrait` for `charl`. This **does not** constitute an automated Information-panel GUI test.

## Questions for maintainer

1. For `name duration resolution`, must **all** terms exist somewhere in the current panel before displaying the **union** of matching rows (current implementation), or should any available matches remain visible if a term is absent?
2. Should every **unquoted word** remain independent, or should recognized multiword field labels be grouped into phrases, even when their words appear across distinct rows?
3. Is a literal `AND` operator required? The current parser treats `AND` as an ordinary search term.

The difference is user-visible: under the current algorithm, `name duration` can display more rows than `name`. These details are **not yet an approved contract**.

## Verification on exact feature head

Head: [`b6b70d3fe1feb15cf2f9ebc9fa282979b8d68634`](https://github.com/BackendArchitectX/DJV/commit/b6b70d3fe1feb15cf2f9ebc9fa282979b8d68634) (6 commits, 9 changed files).

| Evidence | Observed result | Boundary |
| --- | --- | --- |
| [Fork push CI #38123431720](https://github.com/BackendArchitectX/DJV/actions/runs/38123431720) | **12 successful matrix jobs**, release job skipped as expected | Push workflow, not upstream PR workflow |
| Linux CTest | **12/12 CTest targets**, 17 internal `djv-test` classes, no failures, both search classes executed | Not an automated Information-panel GUI test |
| Linux GLES3 CTest | **12/12 targets**, including Recent Files capture | Recent Files capture does not test Information search |
| Windows Debug build + local CTest | Build passed; **1/1 CTest target**, 17 internal classes, zero failures | `DJV_TESTS_NO_GL=ON`: no Windows GUI capture |
| Clean-head GUI | Cross-row search, duplicate Duration, Copy including empty/filtered output, real EXIF, MP4/JPEG and 4-layer EXR changes, Close All, clipboard sentinel | Manual observed final states, not intermediate observer timing |
| [Upstream PR workflow #38123434957](https://github.com/grizzlypeak3d/DJV/actions/runs/38123434957) | **`action_required`, zero jobs executed** | Requires authorized upstream action; **not a pass** |

Real EXIF manual check showed `Exif:FocalLength=50` and `Exif:PhotographicSensitivity=200`; filtered Copy retained those rows. Type query on EXR -> JPEG -> EXR displayed `RGB F16 -> RGB U8 -> RGB F16` at the observed final states.

[Clean-head evidence gallery (11 panels)](https://github.com/BackendArchitectX/DJV/blob/ca8e2c11d4dbeeab4a0ee729c8e80e77340d611e/pr-evidence/919/README.md) | [Complete original description, 21 screenshot links and historical CI/reproduction records](https://github.com/BackendArchitectX/DJV/blob/ca8e2c11d4dbeeab4a0ee729c8e80e77340d611e/pr-evidence/919/pr-919-description-before-review-cleanup-20261012.md)

## Visual evidence — all 21 screenshots

The **21 images below are embedded inline** (not collapsed). All source image paths were verified on the immutable [evidence snapshot](https://github.com/BackendArchitectX/DJV/tree/c5e37fad0475f3f33bb96482b9d3473ccf960cf9/pr-evidence/919). Screenshots 11–21 document clean-head checks from `b6b70d3`; the five composites 17–21 reuse existing clean-head captures, rather than representing new tests. Screenshots 01–10 are historical/supporting evidence and are explicitly distinguished from exact-head proof.

### Clean-head GUI evidence (screenshots 11–21)

#### 11. Real EXIF metadata search

![11 — Real EXIF metadata search](https://raw.githubusercontent.com/BackendArchitectX/DJV/c5e37fad0475f3f33bb96482b9d3473ccf960cf9/pr-evidence/919/11-exif-search-exact-head.webp)

*Exact-head manual check: focallength photographic matches Exif:FocalLength=50 and Exif:PhotographicSensitivity=200.*

#### 12. Copy filtered EXIF metadata

![12 — Copy filtered EXIF metadata](https://raw.githubusercontent.com/BackendArchitectX/DJV/c5e37fad0475f3f33bb96482b9d3473ccf960cf9/pr-evidence/919/12-exif-copy-output.webp)

*Exact-head manual check: copied Metadata includes the displayed EXIF matches.*

#### 13. EXR with four selectable layers

![13 — EXR with four selectable layers](https://raw.githubusercontent.com/BackendArchitectX/DJV/c5e37fad0475f3f33bb96482b9d3473ccf960cf9/pr-evidence/919/13-exr-four-layers.webp)

*Exact-head manual check: Combined, DiffCol, GlossDir and Normal.*

#### 14. EXR → JPEG → EXR pixel type

![14 — EXR → JPEG → EXR pixel type](https://raw.githubusercontent.com/BackendArchitectX/DJV/c5e37fad0475f3f33bb96482b9d3473ccf960cf9/pr-evidence/919/14-pixel-type-exr-jpeg-exr.webp)

*Exact-head manual check: final displayed types RGB F16 → RGB U8 → RGB F16, without observed persistent stale output.*

#### 15. Close All clears Information

![15 — Close All clears Information](https://raw.githubusercontent.com/BackendArchitectX/DJV/c5e37fad0475f3f33bb96482b9d3473ccf960cf9/pr-evidence/919/15-close-all-clears-information.webp)

*Exact-head manual check: zero visible rows after Close All.*

#### 16. Clipboard sentinel

![16 — Clipboard sentinel](https://raw.githubusercontent.com/BackendArchitectX/DJV/c5e37fad0475f3f33bb96482b9d3473ccf960cf9/pr-evidence/919/16-clipboard-sentinel-pass.webp)

*Exact-head manual check: empty Copy replaces a preloaded old clipboard value.*

#### 17. Search term order

![17 — Search term order](https://raw.githubusercontent.com/BackendArchitectX/DJV/c5e37fad0475f3f33bb96482b9d3473ccf960cf9/pr-evidence/919/17-current-head-query-order.webp)

*Composite from earlier clean-head checks: name channels and channels name display the same results.*

#### 18. Duplicate Duration and Copy

![18 — Duplicate Duration and Copy](https://raw.githubusercontent.com/BackendArchitectX/DJV/c5e37fad0475f3f33bb96482b9d3473ccf960cf9/pr-evidence/919/18-current-head-duration-and-copy.webp)

*Composite from earlier clean-head checks: both Duration rows and their copied output.*

#### 19. Missing required term

![19 — Missing required term](https://raw.githubusercontent.com/BackendArchitectX/DJV/c5e37fad0475f3f33bb96482b9d3473ccf960cf9/pr-evidence/919/19-current-head-missing-term-empty.webp)

*Composite from earlier clean-head checks: name nonexistent123 displays no rows and an empty paste.*

#### 20. MP4 → JPEG → MP4 switch

![20 — MP4 → JPEG → MP4 switch](https://raw.githubusercontent.com/BackendArchitectX/DJV/c5e37fad0475f3f33bb96482b9d3473ccf960cf9/pr-evidence/919/20-current-head-mp4-jpeg-mp4.webp)

*Composite from earlier clean-head checks: matches disappear on JPEG and reappear on MP4.*

#### 21. EXIF → MP4 switch

![21 — EXIF → MP4 switch](https://raw.githubusercontent.com/BackendArchitectX/DJV/c5e37fad0475f3f33bb96482b9d3473ccf960cf9/pr-evidence/919/21-current-head-exif-to-mp4.webp)

*Composite from earlier clean-head checks: EXIF query matches JPEG but not MP4.*

### Historical and supporting evidence (screenshots 01–10)

These earlier screenshots document the implementation's progression. Some were captured from a **dirty working tree**, and screenshot 08 is only an inventory of JPEG files. They do **not** independently validate the final feature head.

#### 01. Cross-section search: name channels

![01 — Cross-section search: name channels](https://raw.githubusercontent.com/BackendArchitectX/DJV/c5e37fad0475f3f33bb96482b9d3473ccf960cf9/pr-evidence/919/01-cross-section-name-channels.png)

*Historical capture: File Name and Audio Channels. Not proof of the final clean head.*

#### 02. Duplicate Duration fields

![02 — Duplicate Duration fields](https://raw.githubusercontent.com/BackendArchitectX/DJV/c5e37fad0475f3f33bb96482b9d3473ccf960cf9/pr-evidence/919/02-both-video-and-audio-durations.png)

*Historical capture: Video Duration and Audio Duration are displayed.*

#### 03. Multiword label with Channels

![03 — Multiword label with Channels](https://raw.githubusercontent.com/BackendArchitectX/DJV/c5e37fad0475f3f33bb96482b9d3473ccf960cf9/pr-evidence/919/03-multiword-pixel-aspect-ratio-channels.png)

*Historical capture: Pixel Aspect Ratio and Audio Channels under provisional phrase parsing.*

#### 04. Switch to JPEG without audio

![04 — Switch to JPEG without audio](https://raw.githubusercontent.com/BackendArchitectX/DJV/c5e37fad0475f3f33bb96482b9d3473ccf960cf9/pr-evidence/919/04-media-switch-no-audio-no-results.png)

*Historical capture: name channels has no result on JPEG under current strict presence rule.*

#### 05. Return to MP4

![05 — Return to MP4](https://raw.githubusercontent.com/BackendArchitectX/DJV/c5e37fad0475f3f33bb96482b9d3473ccf960cf9/pr-evidence/919/05-media-switch-return-to-video.png)

*Historical capture: the matching rows reappear after switching back.*

#### 06. Copy filtered rows

![06 — Copy filtered rows](https://raw.githubusercontent.com/BackendArchitectX/DJV/c5e37fad0475f3f33bb96482b9d3473ccf960cf9/pr-evidence/919/06-filtered-clipboard-only-visible-sections.png)

*Historical capture: only nonempty displayed sections appear in the clipboard.*

#### 07. JPEG details

![07 — JPEG details](https://raw.githubusercontent.com/BackendArchitectX/DJV/c5e37fad0475f3f33bb96482b9d3473ccf960cf9/pr-evidence/919/07-image-jpeg-metadata-1920x1080.png)

*Historical capture: 1920 x 1080 JPEG information and no Audio rows.*

#### 08. Supporting JPEG file inventory

![08 — Supporting JPEG file inventory](https://raw.githubusercontent.com/BackendArchitectX/DJV/c5e37fad0475f3f33bb96482b9d3473ccf960cf9/pr-evidence/919/08-downloads-two-jpeg-files-CROPPED.png)

*Cropped filenames in Downloads; not evidence of opening or testing EXIF in DJV.*

#### 09. Working-tree GUI smoke test

![09 — Working-tree GUI smoke test](https://raw.githubusercontent.com/BackendArchitectX/DJV/c5e37fad0475f3f33bb96482b9d3473ccf960cf9/pr-evidence/919/09-local-gui-name-channels-working-tree.png)

*Historical dirty-build screenshot: name channels, not an exact clean-head validation.*

#### 10. Reverse query in working tree

![10 — Reverse query in working tree](https://raw.githubusercontent.com/BackendArchitectX/DJV/c5e37fad0475f3f33bb96482b9d3473ccf960cf9/pr-evidence/919/10-local-gui-channels-name-working-tree.png)

*Historical dirty-build screenshot: channels name, not exact clean-head validation.*

---

## Reproduction

1. Open `sample-5s-360p.mp4`, then Tools -> Information. Search `name channels`; observe File Name and Audio Channels.
2. Search `duration`; observe both Video and Audio Duration. Search `pixel aspect ratio channels` to exercise provisional phrase grouping.
3. Keep `name channels` active and switch to `sample-photo-1920x1080.jpg`; under the current strict presence rule no rows display. Switch back to MP4 and confirm matching rows return.
4. Use Copy and check that only visible nonempty sections are included. Close All and check the prior clipboard value is not retained when Copy is pressed again.

## Review gates and scope

- [x] One consolidated PR (#919); #918 closed as superseded. The workflow correction is in one focused commit and the shared PNG fixture applied once.
- [x] Verified same-head fork CI and local GUI/CTest; evidence branch does not alter feature commits.
- [x] Source-level observer investigation: `InfoWidget::setPlayer` can read a previous video-layer index before the new observer initializes. This order **predates the PR**; bounds fallback exists and the tested final states were correct. No runtime trace or user-visible glitch was established. No unrelated fix is included.
- [ ] Confirm final search semantics with maintainer, then make only necessary code-and-test changes.
- [ ] Obtain upstream PR-workflow authorization and passing tests on the final applicable head.

<details>
<summary>Original #919 feature summary (verbatim)</summary>

> Provisional **implied-AND filtering** for the DJV **Information** panel (related to #316). Search is case-insensitive across field labels and values in File, Video, Audio and Metadata. When every **parsed** term is found somewhere in the current panel, display matching rows; otherwise display none. The exact semantics are pending maintainer feedback, so maintainer feedback is requested before finalizing the search contract.

</details>

<details>
<summary>Original #918 summary and rationale (preserved)</summary>

The independent #918 PR described the following scope before consolidation:

> Fix the PR workflow to ensure tests are executed and prevent false-positive CI results when no tests are discovered.

Its implementation and motivation were:

- Use the existing `ci-linux` configuration to enable `DJV_TESTS`.
- Remove `--rerun-failed` to execute the complete registered test suite.
- Add `--no-tests=error` to fail when no tests are discovered.
- Set a 600-second timeout for test execution.
- A previous successful PR workflow reported **`No tests were found!!!`**, allowing a green test step without executing tests.

Its original validation stated that the workflow change was limited to two lines and that `git diff --check` passed, with Linux CI pending at the time of submission. Subsequently, its separate fork branch [CI run #38093051943](https://github.com/BackendArchitectX/DJV/actions/runs/38093051943) succeeded. This historical run validates **that branch only**, not the new combined commit.

</details>

For auditability, the [immutable pre-cleanup PR description](https://github.com/BackendArchitectX/DJV/blob/ca8e2c11d4dbeeab4a0ee729c8e80e77340d611e/pr-evidence/919/pr-919-description-before-review-cleanup-20261012.md) contains all 21 original screenshot links, earlier test runs, detailed scope discussions and evidence limitations. This replacement is a **description-only** review cleanup; no change to feature code is implied.
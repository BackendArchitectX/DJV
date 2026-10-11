**Current-head Windows GUI and clipboard verification — `b6b70d3`**

- Windows Debug build passed; local CTest passed **1/1 top-level target**, with **17 internal test classes** and zero failures, including both Information search classes.
- Exact-head fork push CI: **12 successful jobs**; Linux and GLES3 CTest **12/12**. The upstream PR workflow is **still `action_required`** and has not executed jobs.
- Verified mixed-case/cross-row implied-AND filtering, reversed terms, missing terms, duplicate Video/Audio Duration rows, filtered and empty Copy, and real EXIF matching (`Exif:FocalLength=50`, `Exif:PhotographicSensitivity=200`).
- Verified four selectable EXR layers, MP4/JPEG/EXIF switching, and EXR → JPEG → EXR pixel type `RGB F16 → RGB U8 → RGB F16`, without persistent stale output or observed crash.
- Close All clears every Information row. A PowerShell clipboard sentinel confirmed Copy with empty information replaces stale clipboard text.

**Six new exact-head screenshots:** [Open verification gallery](https://github.com/BackendArchitectX/DJV/blob/pr-919-evidence/pr-evidence/919/README.md). Images are on the existing evidence branch; **no feature-code commit** was added. The PR description also retains the 10 historical screenshots.

**Evidence limit:** An intermediate media-reference observer update may occur before the new video-layer callback; transient behavior has not been directly instrumented. There is no automated Information-panel GUI capture on Windows (`DJV_TESTS_NO_GL=ON`).

@darbyjohnston Could you please approve the pending [upstream PR workflow](https://github.com/grizzlypeak3d/DJV/actions/runs/38123434957) and confirm whether every unquoted word should be an independent implied-AND term or recognized multiword labels should group first? The current implementation requires all parsed terms to match somewhere in the panel before showing matching rows. Literal `AND` syntax is not implemented. Thank you.
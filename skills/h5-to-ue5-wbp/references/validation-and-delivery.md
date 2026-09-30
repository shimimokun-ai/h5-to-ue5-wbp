# Validation and Delivery

Use this reference after implementation and whenever the requested claim includes runtime appearance, interaction, multi-resolution support, or delivery.

## Evidence ladder

Keep these evidence levels separate:

1. Source inspection: code and assets appear wired correctly.
2. Static checks: naming, references, `git diff --check`, and absence of proof hooks.
3. Runtime source build: native modules/headers or TS/PuerTS generation compile, as applicable to the project.
4. Asset generation/import: the exact WBP/material/texture package is saved.
5. Automation/commandlet: isolated logic or asset checks pass.
6. Headless runtime: the real widget renders and state transitions execute.
7. Manual PIE/user acceptance: input feel and visual quality are accepted by the user.

Do not describe a lower level as proof of a higher one.

## Build and asset generation

- Use the project-pinned engine version and established build script when available.
- For binary-only projects, do not invent a C++ build step. For TS/PuerTS, distinguish `tsc --noEmit`, actual JS generation, WBP typing/registration generation, and engine loading.
- Follow the established editor lifecycle: stop PIE before compiling/saving affected WBP; close the editor for external package replacement or module relinking unless a safe supported path exists. Do not force an editor restart when an in-editor save is supported and the user has already ended PIE.
- Generate only the target asset and confirm the log reports its package path and saved state.
- Check package timestamps and, when relevant, verify Unreal package magic rather than assuming every `.uasset` is valid.
- Scan logs for material errors, failed package loads, fatal errors, and assertions.
- Reload the saved packages in an independent process when practical, then verify widget fields, slots, brush/material references, and imported pixel data. In-memory properties or a successful save call alone do not prove persistence.
- Distinguish operation-phase diagnostics from shutdown diagnostics. An asset operation can finish with zero errors while process teardown reports a PuerTS/container error; report both accurately rather than calling the entire log error-free.

## Headless/offscreen visual proof

When the project/platform support it and the user permits this evidence, launch `UnrealEditor-Cmd` or the game with an offscreen renderer and explicit resolution. Prefer the project's known RHI. Do not convert a screenshot prohibition into offscreen capture permission.

Capture states that prove the requested behavior:

- idle/opened layout;
- opening mid-frame;
- closing mid-frame and fully closed frame;
- scroll overlap under sticky/blurred elements;
- hovered/selected/playing/paused states where automation can reproduce them;
- detail view and back navigation;
- empty or missing-data state;
- 1080p, 1440p, and 4K if multi-resolution clarity is part of the request.

Use full screenshots for layout and cropped screenshots for edge inspection. A crop supplements rather than replaces the full frame.

If only GUI control/visible launch is prohibited, headless proof can be considered within the remaining authorized scope. If screenshots or agent-run visual acceptance are prohibited, use asset/property roundtrip, alpha/channel analysis, material compilation, bounds/hit-region calculations, and logic checks instead. State that runtime visual/input acceptance belongs to the user. User-provided screenshots may guide fixes without authorizing new captures.

## Temporary proof hooks

Command-line-gated timers or camera hooks can make deterministic screenshots possible, but they are temporary test infrastructure.

Rules:

- use a unique, searchable flag name;
- limit the hook to opening the target UI, setting the required state, capturing, and exiting;
- do not alter default behavior without the flag;
- remove the hook and any temporary includes after proof;
- rebuild after removal;
- search the source tree for the flag and screenshot name;
- confirm the proof-owning production file has no residual diff when it should not change.

Never leave auto-click, auto-scroll, auto-open, screenshot, or `quit` behavior in production source.

## Resolution and DPI checks

For each target resolution, inspect:

- text and icon sharpness;
- window placement and authored aspect ratio;
- card/image aspect and crop;
- one-pixel seams, rounded corners, and shadow extent;
- scroll viewport and sticky layers;
- hit regions if manual PIE is available.

High DPI clarity is not the same as OS/PIE window size. Verify UI scale settings, render target/backbuffer, per-window DPI awareness, and physical window dimensions independently when those are in scope.

## Regression checklist

- Entry button still opens the correct destination.
- Page destinations remain mutually exclusive.
- Close/back behavior respects nested detail state.
- Animation disables input appropriately and has no final-frame flash.
- Card selection and player play/pause agree.
- Scroll cards are covered or blurred by floating layers according to the reference.
- No manual WBP values were overwritten by regeneration.
- No project config, PIE size, or unrelated UI assets changed accidentally.
- Save, discard, continue, undo, enabled state, and draft/session ownership still call the real services.
- Dynamic material instances are isolated per thumbnail; display helpers still handle packed textures and unavailable images.
- Scroll fade updates at boundaries and after focus navigation; the scrollbar and nearby controls remain usable.
- Runtime callbacks/tickers are released, and reopen cancels stale close actions.

## Delivery

- Follow the user's delivery scope and the target project's documented process. Commit or push only when requested.
- Identify the source files, editable WBP, and supporting assets needed by the UI. Include generated runtime files when the project requires them, and verify they match the intended source.
- Review an explicit file list and preserve unrelated edits. Exclude unrequested H5 originals, screenshots, logs, caches, and temporary proof artifacts.
- Confirm reported checks actually cover the changed UI. A successful command that skipped the relevant checks is not verification; report skipped checks, failures, and unresolved warnings accurately.
- When publishing or submitting changes, verify the intended destination received the current files. Report delivery status separately from build, runtime, and visual acceptance.

When editor shutdown asks to save materials or WBP changed by the task, inspect the dirty-package list and ownership. Save intended packages through the supported editor path; do not advise saving every package or discarding all changes without knowing what the dialog contains.

## Final report

Include:

- editable WBP path and native/runtime controller path;
- generated/imported support assets;
- implemented states and intentional omissions;
- adjustable Designer properties;
- build command/result;
- asset generation result;
- headless resolutions and screenshot links;
- manual PIE status;
- submission or publication status when requested, and any unrelated changes left untouched.

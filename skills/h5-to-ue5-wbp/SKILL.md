---
name: h5-to-ue5-wbp
description: Reproduce an HTML/CSS/JavaScript UI reference as an editable Unreal Engine 5 UMG Widget Blueprint, preserving project behavior and matching assets, layout, interaction states, DPI, and animation. Use for native H5-to-WBP conversion or refinement; verify at the user's requested evidence level. Web embedding requires an explicit request.
---

# H5 to UE5 WBP

Turn an H5 reference into a native, editable UE5 UMG implementation whose visual hierarchy, states, and behavior remain maintainable in the target project. Treat the H5 as a specification, not as runtime web content.

## Start with the requested evidence level

- If the user asks only whether the conversion is feasible or asks for an explanation, inspect the H5 and current UE implementation read-only. Do not create assets or edit source.
- If the user asks to build or update it, trace the existing widget entry point, ownership, data flow, and close/back behavior before editing.
- Respect explicit restrictions on computer use, screenshots, visible launches, and PIE. A ban on screenshots also covers headless/offscreen capture; it is not permission to switch capture methods. When the user owns visual acceptance, use source, asset, log, and numerical checks and report that boundary.
- Separate source implementation, saved `.uasset`, material/build checks, runtime interaction, and user visual acceptance. Report only the evidence collected; one level does not prove another.
- Preserve unrelated dirty-worktree changes. Never broadly stage, reset, or regenerate unrelated widgets.

## Inspect both sides before choosing an architecture

1. Read the actual HTML, CSS, and JavaScript, including linked assets, CSS variables, media queries, hover/selected states, sticky regions, overflow rules, shadows, blur, and animations. A screenshot alone is insufficient when source is available.
2. Inspect the target UE version, project UI scale settings, current WBP parent class, existing named widgets, construction path, input bindings, data providers, and whether the WBP Designer tree is already artist-edited.
3. Record a compact design specification from the final CSS cascade: authored viewport, bounds, spacing, typography, colors, radii, image fit, layers, scroll ownership, per-control interaction states, and animation timings. Shared CSS defaults may be overridden by control-specific files.
4. Decide what must be editable in WBP and what belongs in runtime logic. Do not silently substitute a runtime-only widget tree when the user explicitly wants a fine-tunable user control.

Read [references/h5-umg-mapping.md](references/h5-umg-mapping.md) for the layout mapping, editable-WBP architectures, and asset strategy before implementing a new conversion or materially restructuring an existing one.

## Preserve editability and one source of truth

Prefer this shape when the UI has meaningful runtime logic:

- The project's existing controller (C++, Blueprint, or TS/PuerTS) owns data, state transitions, and event handlers. Editable WBP does not require a new C++ class; a binary-only project may have no native source/build capability.
- A real Widget Blueprint serializes the presentation tree and exposes useful layout/style parameters.
- Runtime code resolves stable named widgets and binds actions without replacing the serialized tree.
- An Editor command, commandlet, or other deterministic generator may create the initial WBP tree and imported support assets.

If a generator owns the Designer tree, treat regeneration as destructive to manual WBP edits. Before regenerating:

- inspect whether the asset has artist changes;
- move accepted manual values into the generator/source or deliberately preserve the asset;
- regenerate only the named WBP;
- reload or reopen the asset after generation before judging it.

Blueprint-only implementation is valid when the project has no suitable C++/Editor path and the interaction remains maintainable. Runtime-only construction is valid only when WBP editability is not a requirement.

## Implement in layers

1. Establish one authored design coordinate system and one viewport fitting layer. Avoid independent scaling of nested regions.
2. Build the structural hierarchy first: window, header, content columns, scroll regions, sticky/floating layers, overlays, and modal/close affordances.
3. Import or reuse images, icons, fonts, and materials. Preserve original source assets outside cooked Content when the project convention calls for it.
4. Match typography, fills, corner radii, shadows, blur, image cropping, and clipping. Preserve capsule end geometry, fit glyphs without distortion, and inspect alpha-visible bounds when icons look off-center. CSS font pixels may need conversion through the target engine's font/DPI path.
5. Add the states required by the H5 for each control family. Do not apply one hover/wave animation to every button; solid action buttons may have only fill, press, and rebound effects.
6. Connect actual project data and gameplay services. H5 sample content may guide appearance but must not replace real project state unless the user asks for sample data.
7. Add transitions to one coherent visual root when the whole window should move or fade together. Disable hit testing during close and collapse only after the transparent final frame.
8. Expose the values the user is likely to tune—durations, distances, radii, colors, opacity, and layout offsets—as WBP defaults or structured style properties when practical.

For an existing editor, map every control to the real service operation before adding behavior. Use H5 icons for category/actions and project assets for actual items when requested. Preserve save/cancel, draft ownership, missing-data states, and input routing; browser demo export or localStorage is not a replacement for game persistence. Read the integration guidance in [references/h5-umg-mapping.md](references/h5-umg-mapping.md).

For rounded images, blur, clipping, z-order, button padding, retainer composition, and animation edge cases, read [references/umg-visual-pitfalls.md](references/umg-visual-pitfalls.md) before finalizing or when a screenshot shows seams, fringes, stretching, gaps, or flashes.

## Validate proportionally to the claim

Choose checks that support the requested claim and the available project capabilities:

1. Run source/static checks and `git diff --check`.
2. Run the established runtime build (including TS-to-JS where applicable). Build the pinned Unreal Editor target only when native changes or the project workflow require it and the environment supports it.
3. Run the specific asset generator/import path if used and confirm the named package was saved.
4. Inspect logs for material compilation errors, package failures, fatal errors, and assertions.
5. When runtime visual verification is permitted and in scope, render meaningful states at the target resolutions. Otherwise validate saved properties, alpha data, material compilation, geometry, and state logic; leave visual acceptance to the user.
6. Verify relevant state-changing interactions at the available evidence level. Simulated-engine logic tests do not establish real Slate input, rendering, or gameplay acceptance.
7. Remove temporary proof hooks, rebuild, and verify no proof flags or test-only behavior remain.
8. Report what was not tested, especially manual PIE input or user visual acceptance.

Read [references/validation-and-delivery.md](references/validation-and-delivery.md) for headless proof patterns, resolution checks, artifact inspection, and narrow Git delivery.

## Guardrails

- Do not use a Web Browser widget merely because the reference is HTML.
- Do not declare pixel parity from one screenshot; compare layout, state, and behavior at the intended resolutions.
- Do not solve 4K clarity by changing the user’s chosen standalone window size. DPI/render clarity and OS window dimensions are separate concerns.
- Do not flatten an editable UI into a single raster image unless the user explicitly requests that tradeoff.
- Do not overwrite accepted WBP edits by rerunning a generator without reconciling the source of truth.
- Do not leave temporary camera, screenshot, command-line, timer, or auto-click hooks in production code.
- Commit or push only when requested. Follow the target project's documented submission process, review an explicit file list, and preserve unrelated work.

## Handoff

Lead with the implemented outcome. Summarize:

- where the editable WBP and controlling source live;
- which H5 behaviors were reproduced and which were intentionally omitted;
- which values are exposed for Designer tuning;
- build, generation, runtime screenshot, and manual PIE evidence separately;
- target resolutions tested;
- remaining dirty files, branch/commit state, and whether anything was pushed.

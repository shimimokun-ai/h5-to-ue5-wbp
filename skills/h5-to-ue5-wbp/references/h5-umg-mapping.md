# H5 to UMG Mapping and Architecture

Use this reference when implementing a new H5-to-WBP conversion or materially restructuring one.

## Extract a design specification from the H5

Read the source and capture these values before building UMG:

- authored page/viewport size and central window bounds;
- fixed, absolute, grid, flex, and sticky relationships;
- margins, gaps, padding, min/max sizes, and overflow ownership;
- colors including alpha, gradients, outlines, shadows, and backdrop blur;
- typography: family, weight, size, line height, letter spacing, wrapping, ellipsis;
- image `object-fit`, `object-position`, aspect ratio, and rounded clipping;
- default, hover, pressed, selected, playing, paused, disabled, loading, and empty states;
- transition duration, delay, easing, translation, scale, opacity, and transform origin;
- data fields and whether content is sample-only or reflects real project data.

Use browser rendering as evidence when permitted, but treat the HTML/CSS/JS as authoritative for exact values. If a local `file://` page is blocked, read it directly or serve it from a local HTTP server when permitted. Respect user restrictions on browser control and screenshots.

Resolve the final CSS cascade before transferring values. Base button rules, surface rules, palette rules, and control-specific overrides can have different dimensions and animations. Build a small state table for each relevant control family: resting fill, hover/focus fill, wave eligibility, selection layer, press transform, release curve, disabled opacity, and transition duration. Record transform origin and clipping separately from layout size.

## Common layout mapping

| H5/CSS concept | Typical UMG equivalent | Notes |
|---|---|---|
| authored viewport | `SizeBox` inside `ScaleBox` | Use one design stage and scale it once. |
| absolute positioning | `CanvasPanel` + `CanvasPanelSlot` | Best for reference-exact overlay windows. |
| flex row/column | `HorizontalBox` / `VerticalBox` | Set fill/auto rules and slot padding explicitly. |
| CSS grid/cards | `WrapBox`, `UniformGridPanel`, or explicit canvas | Choose based on runtime item count and responsive behavior. |
| overflow scroll | `ScrollBox` | Identify the one widget that owns scrolling. |
| sticky/fixed bar | sibling overlay/canvas layer above the scroll content | Do not place it inside the scrolling subtree unless the H5 does. |
| `border-radius` | rounded `FSlateBrush`, material mask, or retainer mask | Native blur and image masks need separate attention. |
| `object-fit: cover` | custom UI material with aspect-aware UV crop | `Image` stretching alone may distort. |
| box shadow | dedicated soft material/image behind the card | Avoid offset opaque rounded rectangles. |
| `backdrop-filter: blur()` | `UBackgroundBlur` | Blur only samples content below it in the same Slate composition. |
| hover/active class | button events plus explicit presentation state | Avoid relying on default UE button styling. |
| CSS transition | render transform/opacity advanced in `NativeTick`, timeline, or WBP animation | Apply to a coherent root for whole-window motion. |
| z-index | `CanvasPanelSlot::SetZOrder` and tree order | Confirm both parent hierarchy and local z-order. |

## Choosing the WBP architecture

### Serialized Designer tree plus native behavior

Use when the user wants a real, fine-tunable WBP and the page has non-trivial data or gameplay behavior.

Recommended responsibilities:

- Existing project controller (native widget, Blueprint, or TS/PuerTS panel): data loading, gameplay calls, state machine, event handlers, timers. Reuse the established route instead of requiring a new C++ layer.
- WBP Designer tree: hierarchy, slots, sizes, colors, materials, editable defaults, asset references.
- Runtime binding: resolve widgets by stable names or `BindWidget`, then bind delegates without rebuilding the tree.
- Editor generator/commandlet: create the first serialized tree, import deterministic support assets, and save the package.

The generator is not the runtime UI. Its output must be a valid Widget Blueprint package, and the runtime should use the serialized tree.

### Blueprint-only

Use when logic is modest, designers own the behavior, or the project lacks a safe native/editor extension path. Keep complex data/service logic outside the presentation graph.

### Runtime-only native construction

Use only when the user does not require Designer editability, the project already uses that convention, or the tree is fully dynamic and a serialized layout provides little value. Explain the editability tradeoff.

## Generator and manual-edit ownership

A deterministic generator is useful for large H5 layouts because it makes dimensions and named widgets reviewable. It is also capable of overwriting manual WBP changes.

Before every regeneration:

1. identify whether the WBP package changed after the last generated source change;
2. record accepted Designer edits;
3. copy those values into generator defaults or another persistent style source;
4. regenerate only the target asset;
5. verify the saved package and reopen/reload it;
6. compare the result with the accepted presentation.

Do not regenerate unrelated Widget Blueprints or all UI assets as a convenience.

## Asset strategy

- Reuse existing project assets when they match the reference and licensing is clear.
- Preserve source PNG/JPG/SVG/font files according to project convention; cooked UI should reference imported Unreal assets, not external absolute paths.
- Use deterministic names grouped by the feature, such as `T_`, `M_`, `MI_`, `WBP_`.
- Import icons as alpha-safe UI textures and check compression/filtering.
- Separate a surface's alpha silhouette from its tint. A reusable white tint mask needs white RGB even in fully transparent texels; a black silhouette multiplied by a brush tint remains black. Preserve the correct alpha convention when rasterizing SVGs.
- Where specified, use H5 category/action icons but retain the project's actual item thumbnails, masks, and data. A project source texture may encode material channels rather than a display-ready picture; inspect its channels before choosing a thumbnail path.
- Use aspect-aware cover materials for variable source sizes.
- Use analytic/material shadows when large soft shadows would scale poorly as raster images.
- Verify `.uasset` files are real packages rather than Git LFS pointer text before runtime claims.

## Data and interaction mapping

Keep sample presentation separate from real behavior:

- H5 sample cards can define number of visible slots, visual density, and empty-state appearance.
- UE runtime collections should come from the project’s real model or subsystem.
- Preserve real missing/empty states instead of inventing H5 sample memberships.
- Split click targets only when the behavior is discoverable—for example, cover for play/pause and text body for details. Add tooltips or visible cues if the split is subtle.
- Make back behavior stateful: detail → collection → close, not detail → immediate close.
- Keep play/pause state synchronized between cards, detail view, and persistent player.

## Adapting an existing editor without changing its features

- Map category, style, color, enable, parameter, reset, preview, paint, save, and cancel controls to their current service calls. Stable widget identities must not depend on visible text after text is replaced with icons.
- Keep data ownership and draft/session checks. Do not enable unsupported H5 sample features, manufacture selectable items, or replace the real paint editor with the browser demonstration.
- Reuse the current input mode and focus system. In a Slate UI-only route, PlayerController key bindings may not receive input; use the project's widget preview-key path and explicit navigation where needed. Keep callbacks on the live widget and release them on unmount. Never cache world objects or session callbacks in a global class registry.
- For undo, snapshot the actual applied appearance, equipped slots, and committed painting as required by the project. Ignore no-op changes, bound history, and retain the current state/history on a failed restore. Category changes are presentation state, not necessarily undoable edits.
- Keep save/discard/continue semantics distinct. If a shared message service only supports two choices, inspect extension points and the project's rules before replacing a three-choice H5 dialog. Preserve the user's requested behavior and explicitly document an unresolved framework mismatch; do not claim the integration is fully compliant while it remains.
- Restore each category's scroll position and synchronize focus after rebuilding a grid or color palette. Enter/leave animations must handle reopen during close without allowing a stale callback to collapse the reopened panel.
- Hide or disable unsupported controls with accurate empty states. Do not interpret a style-only request as authorization to add new gameplay capabilities.

## DPI and responsive behavior

- Distinguish authored Slate units, UI scale curve, render target resolution, OS DPI awareness, and standalone window dimensions.
- Prefer a single design stage (`1920×1080`, `1440×810`, or the actual H5 size) fitted once to the viewport.
- Validate 1080p, 1440p, and 4K when the UI is expected to support them.
- Do not introduce nested ScaleBoxes that independently resample text and icons.
- Do not change the user’s standalone PIE window defaults solely to make screenshots sharper.

For typography, inspect the target engine's font sizing and DPI calculation before copying CSS pixel values into `FSlateFontInfo::Size`. A 96-DPI path that converts font points with 96/72 can require `CSS px * 72 / 96`; other engine versions/settings may differ. Preserve the H5 font's metrics, weight, line height, centering, and padding rather than applying a universal conversion blindly.

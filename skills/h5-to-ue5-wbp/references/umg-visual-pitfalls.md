# UMG Visual Pitfalls

Use this reference when implementing or debugging blur, rounded corners, images, clipping, animation, layer order, or small seams.

## Black surfaces and dark alpha fringes

Distinguish an entirely black surface from a narrow dark fringe:

- For a tintable surface, inspect RGB and alpha independently. Rasterized black SVG geometry tinted orange in Slate is still black; use a white silhouette with the original alpha if the intended color comes from the brush/material.
- Straight-alpha textures with black RGB in transparent texels can produce dark edges under bilinear filtering, fractional positions, scale, or animation. For a uniformly white tint mask, set **all** texels' RGB to white, including alpha-zero texels, and preserve alpha exactly. For multicolor art, extend neighboring edge colors into transparent texels rather than bleaching the asset.
- Confirm straight versus premultiplied alpha throughout rasterization, import, material output, and blend mode. Do not premultiply twice or mix conventions to conceal an edge defect.
- Check clamp addressing, UI compression, mip policy, and filtering for the target usage. Bilinear filtering is useful for smooth edges once transparent RGB is correct; disabling filtering is not a general fringe repair.
- In an affected engine build, a RoundedBox shader may interpolate fill and outline colors at its edge even with outline thickness zero. Inspect that build's Slate shader/brush settings when a border remains. A compatible outline color or a UI material with constant RGB and alpha-only antialiasing can avoid the unwanted dark mixture.

For a texture-only repair, compare source and saved/reloaded pixels: alpha should remain unchanged, intended RGB corrections should be measurable, and unrelated textures should retain their hashes. Numerical roundtrip verifies asset data, not runtime appearance.

## Capsules must retain end geometry

Do not resize a complete capsule independently in X and Y to fit every control. That turns circular ends into ellipses and can flatten shadows or create inward notches.

- Prefer the H5's actual surface variant for the target control: full pill, half-left/right adjustment surface, small chip, or vertical preset panel may have different geometry.
- For variable length, preserve the end sections and extend only the straight middle; use suitable nine-slice margins or an analytic material with target-size parameters. Align raster cuts to the source pixel grid to avoid translucent seams.
- Scale the icon uniformly to fit its content rectangle, leaving padding as necessary. Keep the clickable area at its intended size.
- Give background, selection, hover mask, and shadow compatible silhouettes and dimensions. A correctly shaped normal surface can still be distorted by an old hover/selected brush.
- Compare color-chip **inner** and outer dimensions with final CSS. A color rectangle can legitimately stretch to its specified inner width while its rounded end geometry remains fixed; shrinking the whole chip to preserve an image's unrelated aspect can make the swatch too small.

## Icon centering and texture-channel semantics

Geometric bounds do not necessarily center the visible glyph. Measure nonzero-alpha bounds or the visible path inside the SVG viewBox, then center that shape within the H5 content area. Preserve intentional optical offsets from the reference. Check both selected and unselected icons; neither should shift when toggled.

For black-background/white-shape project textures, determine whether they are masks before treating the background as an import failure:

- Use a thumbnail-only UI material to map the verified mask channel to alpha and the desired display color to RGB. Bind each source texture to its own dynamic material instance so one item cannot change every thumbnail.
- Packed eye/mouth textures, display-ready attachments, and face icons may need different paths. Preserve existing project preview helpers and fallbacks rather than applying one mask rule globally.
- Faint gray masks can encode shading intensity rather than icon coverage. With sRGB sampling, a channel value around 57-68/255 becomes only about 0.041-0.058 in linear space. If that channel directly drives alpha, the glyph looks almost transparent.
- Measure core, background, and edge values before adding per-category gain/normalization such as `saturate(mask * gain)`. Keep the default unchanged for other masks and preserve antialiased edges. A gain used for one wrinkle dataset is not a general default.

Do not change character material parameters or source textures to repair the catalog's appearance. Also inspect parent render opacity, disabled Slate effects, motion state, and retainer composition before attributing every faint icon to texture data.

## Scrollbar width and circular ends

Inspect the complete scrollbar style, not only `ScrollbarThickness`:

- Hidden `NoDraw` brushes can still have default `ImageSize` (for example 32 by 32) that affects layout/minimum size. Explicitly zero unused images and set the sizes of track and normal/hovered/dragged thumb brushes consistently.
- Vertical thumb length changes with the visible fraction of content. A fixed square texture stretched to a long thumb does not preserve a true semicircular end. The radius should be half the **actual rendered width**, independent of thumb length and UI scale.
- Prefer an engine-supported rounded brush if its rendering matches the reference. If it still produces dark borders or distorted ends, use an aspect-aware distance-field UI material; derive rendered aspect from verified size parameters or screen-space UV derivatives and antialias alpha while keeping RGB constant.
- Set outline width/color deliberately. Match the H5's low-opacity track rather than adding a solid border. Keep normal, hover, and dragged states consistent.
- Preserve native drag/scroll behavior and the hit target. Apply catalog fade to items, with a separate scrollbar or an excluded scrollbar strip.

Check multiple thumb lengths and scales numerically when screenshots are disallowed. Do not report success based only on a nominal radius property.

## Scroll-edge feathering over a real background

An opaque gradient painted in the page's flat color will hide a patterned or animated background instead of revealing it. Fade the catalog's composed alpha through a retainer/effect material and disable incompatible native scroll shadows.

Compute fades from the current offset, end offset, and viewport height. For a reference fade distance `D` in design units:

```text
end = max(0, endOffset)
offset = clamp(scrollOffset, 0, end)
topFade = min(D, offset) / viewportHeight
bottomFade = min(D, end - offset) / viewportHeight
```

Guard empty/zero geometry. At the top, no top fade; at the bottom, no bottom fade; for content fitting one screen, neither edge fades. Recompute after wheel inertia, category offset restoration, keyboard scroll-into-view, and content/viewport changes. Respect the retainer's update rate so it does not reduce existing animation smoothness.

Match the material's blend convention: with premultiplied-alpha composition, reduce RGB and alpha together. With a straight-alpha path, follow its expected contract. Verify the chosen path rather than universally multiplying RGB by opacity.

## Character preview framing and boundary sampling

Treat the preview slot, image rect, capture camera, render target, and drag hit region as separate causes:

- For modest enlargement, tune image size and capture framing without changing gameplay actor scale. Calculate headroom, the intended body crop, and clearance from nearby controls. Keep the drag hit region aligned with the preview image so it cannot intercept neighboring buttons.
- For an orthographic capture, visible vertical limits depend on target height and orthographic width/aspect. Preserve the intended upper bound when changing size and adjust camera target deliberately; do not guess by changing unrelated character transforms.
- Moving a reset-view control above the head must preserve its event and leave room for hover lift, shadow, and other UI hit targets.
- If the preview should touch the design-stage bottom, use bottom anchoring/alignment or the equivalent responsive layout. Inspect both image placement and transparent capture padding; moving only the parent does not always place the visible character edge correctly.
- A thin line above the head can come from render-target wrap filtering sampling the bottom row, rather than a gap in the model. Inspect UVs, addressing, alpha extraction/inversion, and capture borders. A clamped UI sampler can fix this; confirm the exact enum/API name in the pinned engine version.
- When a shared preview material serves other screens, prefer a local instance/variant when possible, preserving its texture parameter and alpha convention. Bind the current render target to the material actually displayed by the image.

## Button content can be inset even when the style padding is zero

`FButtonStyle::NormalPadding` and `PressedPadding` are not the same as the child `UButtonSlot::Padding`.

In UE 5.x, a `UButtonSlot` may default to horizontal/vertical content padding (for example 4 px and 2 px) and centered alignment. A full-bleed cover inside a transparent button can therefore reveal the card background around its edges.

For a full-bleed button child, explicitly set:

- `UButtonSlot::SetPadding(FMargin(0))`
- horizontal alignment to `HAlign_Fill`
- vertical alignment to `VAlign_Fill`

Inspect both the style and the content slot before changing image size or adding overscan.

## `FVector4(Value)` is not four equal radii

Do not assume `FVector4(20.f)` produces `(20,20,20,20)`. In relevant UE constructors, omitted components use their own defaults. Set four corner values explicitly:

```cpp
FVector4(20.f, 20.f, 20.f, 20.f)
```

Check the actual engine version’s constructor before generalizing.

## Rounded visual shape and blur mask may need different radii

`UBackgroundBlur` clips its sampling region using its own corner radius. A blur mask and a foreground rounded border with the same nominal radius can still differ by one or two antialiased pixels, revealing a sharp scrolled item at the corner.

When evidence shows this mismatch:

- keep the foreground card at the intended visual radius;
- use a slightly broader blur coverage mask behind it, often represented by a smaller blur corner radius;
- keep the blur underneath the foreground card and above scroll content;
- verify no rectangular blur patch leaks outside the visible card.

Do not add an opaque masking rectangle as a substitute for correct blur coverage and z-order.

## Backdrop blur only affects content below it

`UBackgroundBlur` samples previously rendered content. It cannot blur a sibling or child rendered above it. Verify:

- parent hierarchy;
- local z-order within each canvas;
- whether the scroll cards and blur share a composition path;
- whether a Retainer Box changes the composition boundary.

Player controls and cover art that belong to the glass foreground should remain sharp. Scrolled content behind the player should blur.

## Retainer opacity can expose internal seams

Applying partial opacity directly to a parent containing a `RetainerBox` can affect both the offscreen content pass and composite pass, sometimes exposing boundaries between overlapping child layers.

For a whole-window fade:

- composite the accepted presentation first;
- apply transition opacity after composition, commonly through an effect material parameter;
- keep child layout and internal opacity stable during the window transition;
- include shadow/backdrop in the same transition intentionally.

If no retainer is involved, root render opacity may be sufficient; verify a mid-fade screenshot.

## Close animations can flash on their last frame

A common failure is restoring the resting transform/opacity before the widget is actually collapsed, producing a visible one-frame flash.

Safer sequence:

1. stop hit testing without applying a disabled draw effect;
2. animate from the currently rendered transform and opacity;
3. render the fully transparent final frame;
4. collapse on the next game tick;
5. only after collapse, restore the resting transform and opacity.

If close occurs during opening, use the current rendered transform/opacity as the close start.

## Image fit and rounded crop

- Stretching an `Image` to a target rectangle can distort source art.
- An aspect-aware material should compute cover-crop UVs from target dimensions and source texture aspect.
- Round only the corners the reference rounds. A cover joined to a text body often has rounded top corners and square bottom corners.
- Pass actual design dimensions to the material so the radius is expressed in meaningful UI units.
- Check texture alpha, sampler type, compression, and filtering when a fringe persists.

## Layering and clipping

- A `CanvasPanelSlot` z-order applies within its parent; it does not override a different parent’s composition order.
- Use clipping only where content must be cut. Do not crop a card at a player boundary when the H5 instead lets the higher player layer cover it.
- Sticky bars should usually be siblings above the scroll content.
- Shadows belong behind the visual card and may need a larger extent than the card itself.
- A mask added to hide a z-order problem often creates new corner and blur problems.

## Hover and selected appearance

- UE default button brushes can add gray pills, pressed offsets, padding, or darkening that are absent from the H5.
- Build explicit normal/hovered/pressed/disabled brushes.
- For text-only tabs, remove the background brush in all states and change typography/color deliberately.
- Keep selected, hovered, playing, and paused as distinct states. Do not leave a playing highlight active after pause if the H5 clears it.
- Derive wave/ripple eligibility per control family from final CSS/JS. A solid orange exit/done action can have a hover fill and press/rebound without a wave overlay. Selection and keyboard preselection may also suppress or alter waves.
- When rendering separate selected/unselected icons, make their visibility mutually exclusive after animation updates; a transform tween must not restore both to visible.
- Avoid applying enabled opacity both to a parent control and its glyph unless the H5 explicitly does so; compounded alpha can make available items look disabled.

## Pixel seams and fractional layout

- Fractional Slate positions can be valid, but adjacent translucent layers may reveal seams after DPI scaling.
- First identify which layer owns the seam; do not label every 1 px line as an outline.
- Prefer one continuous surface over multiple adjacent fills when the reference is visually continuous.
- Validate at more than one scale because a seam may appear only at a specific DPI.

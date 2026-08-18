---
name: design-macos-app-icon
description: Design, revise, export, integrate, and validate original macOS app icons using Adobe Illustrator scripting and Apple Icon Composer. Use when asked to 设计或重做 Mac 应用图标, create layered SVG assets, automate Illustrator icon construction, build or update a .icon file, integrate it into Xcode, or perform small-size and appearance QA. Treat SF Symbols only as interface-icon resources and enforce Apple's prohibition against using symbols or confusingly similar images in app icons, logos, or trademarks.
---

# Design macOS App Icon

## Required reading

Read [references/workflow.md](references/workflow.md) completely before planning or modifying an icon. Follow its source-of-truth hierarchy, stage gates, file layout, validation matrix, and recovery rules.

## Operating model

Own the design-tool work. Do not require the user to learn Illustrator or Icon Composer.

1. Ask for only the missing product meaning that would materially change the icon. Prefer one high-value question at a time.
2. Produce three concise, original concept directions when no direction exists. Recommend one and explain the tradeoff at icon scale.
3. Use Adobe Illustrator's JSX/AppleScript interface for deterministic vector construction, layer management, saving, and export.
4. Use the minimum necessary Icon Composer GUI interaction only after the layered artwork is ready. No public Icon Composer scripting interface is assumed.
5. Show rendered previews and obtain visual feedback. Never claim visual success from SVG/XML inspection alone.

## Non-negotiable boundaries

- Never place, trace, redraw, or closely imitate an SF Symbol in an app icon, logo, or trademark-related asset. Apple prohibits symbols and confusingly similar images in those uses.
- Use SF Symbols only for in-app interface glyphs and related vocabulary checks. Do not treat an exported symbol as app-icon source art.
- Create original silhouettes and geometry. Do not trace third-party app icons or brand marks.
- Create a new Illustrator document for each icon version. Do not alter the user's open document or overwrite an earlier version unless explicitly asked.
- Use explicit absolute input and output paths. Refuse implicit home-directory, wildcard, or unresolved-variable targets.
- Before sending Apple Events to Illustrator, explain the action and request the required system approval. A first run may also require macOS Automation permission.
- Keep source artwork flat and controllable. Leave platform masks, blur, shadow, specular highlight, refraction, translucency, and similar material effects to Icon Composer.
- Do not pre-mask the artwork as a rounded rectangle or circle.
- Use 1024 x 1024 for iPhone, iPad, and Mac artwork unless the current Apple template for the target says otherwise.
- Keep all exported layers on the same full canvas and coordinate system. Prefer SVG; use transparent PNG only for content that SVG cannot preserve correctly.
- Convert text to outlines before SVG export.

## Required stage gates

Do not skip these gates:

1. **Brief gate** — state the app meaning, audience, primary metaphor, desired character, exclusions, and platform scope.
2. **Concept gate** — compare three black-and-white silhouettes at small size; select one direction before polishing.
3. **Specification gate** — record the canvas, geometry, palette, layer order, names, and export targets in a reproducible icon specification.
4. **Vector gate** — create a new 1024 x 1024 RGB Illustrator document with stable object and layer names.
5. **Preview gate** — render and inspect at least 1024, 256, 128, 64, 32, and 16 px before Icon Composer.
6. **Composition gate** — import numbered layers from back to front, keep the composition to at most four Icon Composer groups, and tune available appearance modes.
7. **Build gate** — when integrating with an app, verify the `.icon` target resource, build setting, generated `.icns`, `Assets.car`, and compiled Info.plist icon keys.
8. **Delivery gate** — preserve editable source, layered exports, previews, Icon Composer source, integration notes, and known uncertainties.

## Illustrator interface route

Prefer direct scripting over UI automation. Generate task-specific JSX in a versioned work directory and invoke it on macOS with an explicit absolute path, for example:

```sh
osascript -e 'tell application id "com.adobe.illustrator" to do javascript (POSIX file "/absolute/path/build_icon.jsx")'
```

Treat Illustrator as a visible desktop process, not a headless renderer. Check the script result and exported files. If an OS prompt, document dialog, missing font, or unsupported SVG feature blocks the run, stop and report the exact blocker instead of clicking through blindly.

## Visual review contract

For every meaningful revision:

1. Export a large PNG and a small-size contact sheet.
2. Inspect the actual pixels.
3. Report one recommendation and the most important remaining weakness.
4. Ask the user for a single focused choice or adjustment.

Reject a version when the 32 px silhouette is ambiguous, important shapes merge, contrast collapses in an appearance mode, or the visual center feels displaced even when geometry is mathematically centered.

## Completion response

Report:

- the selected concept and why it survived small-size review;
- files created or modified;
- Illustrator, SVG, Icon Composer, Xcode, and visual checks actually performed;
- checks not performed and why;
- any manual approval or product decision still required.

Do not claim completion while the icon has only been drawn at 1024 px or only previewed inside Illustrator.

# H5 to UE5 WBP

[简体中文](README.md) | **English**

A Codex skill for recreating HTML/CSS/JavaScript UI references as native, editable Unreal Engine 5 UMG Widget Blueprints.

Here, **H5** refers to an HTML/CSS/JavaScript interface used as a reference for appearance and behavior. The skill covers layout, assets, interaction states, project integration, DPI, animation, and validation based on the evidence collected.

## Use cases

- Recreate a UE5 interface from H5 source while keeping its layout and styling editable in the WBP Designer.
- Refine an existing UMG interface to match a reference and diagnose rounded corners, alpha fringes, blur, scrolling, DPI, and animation issues.
- Connect the interface to the project's existing C++, Blueprint, or TS/PuerTS data and interaction logic.
- Report source inspection, compilation, saved assets, runtime checks, and user visual acceptance as separate levels of evidence.

This repository provides instructions and reference material for Codex. It is not an Unreal Engine plugin or a one-click converter that works without project-specific adaptation. Native UMG is the default; embedding a Web Browser widget is considered only when explicitly requested.

## Installation

### Install through Codex

Enter the following in Codex:

```text
Use $skill-installer to install the skill from
https://github.com/shimimokun-ai/h5-to-ue5-wbp.
Path within the repository: skills/h5-to-ue5-wbp
```

### Manual installation

Download or clone this repository, then copy the entire `skills/h5-to-ue5-wbp` folder into your Codex skills directory.

- Default location: `~/.codex/skills/h5-to-ue5-wbp`
- Default Windows location: `%USERPROFILE%\.codex\skills\h5-to-ue5-wbp`
- If `CODEX_HOME` is set: use `skills/h5-to-ue5-wbp` inside that directory.

Keep the `agents` and `references` subdirectories. If a skill with the same name already exists at the destination, review any local changes before deciding whether to update it.

## Usage examples

After installation, work in the target Unreal project and give Codex the H5 source location, the target UI entry point, and your validation requirements. For example:

```text
Use $h5-to-ue5-wbp to recreate the H5 interface in docs/ui-reference
as an editable, native UE5 Widget Blueprint. First inspect the existing
UI entry point and data flow. Preserve the project's current behavior
and explain where the layout and styling can be adjusted.
```

For an assessment without implementation:

```text
Use $h5-to-ue5-wbp to inspect this H5 reference and the current Unreal UI
in read-only mode. Assess the conversion approach, effort, and limitations.
Do not change code or assets yet.
```

If you want to handle runtime acceptance yourself, add:

```text
For this task, perform only source, build, and asset checks.
Do not launch Play In Editor (PIE) or capture screenshots.
Report what has been verified and what still needs my acceptance separately.
```

## File structure

```text
skills/h5-to-ue5-wbp/
├── SKILL.md
├── agents/
│   └── openai.yaml
└── references/
    ├── h5-umg-mapping.md
    ├── umg-visual-pitfalls.md
    └── validation-and-delivery.md
```

- [SKILL.md](skills/h5-to-ue5-wbp/SKILL.md): scope, core workflow, and handoff requirements.
- [H5 / UMG mapping](skills/h5-to-ue5-wbp/references/h5-umg-mapping.md): layout, architecture, assets, and project integration.
- [UMG visual pitfalls](skills/h5-to-ue5-wbp/references/umg-visual-pitfalls.md): rounded corners, transparency, blur, scrolling, animation, and related details.
- [Validation and delivery](skills/h5-to-ue5-wbp/references/validation-and-delivery.md): levels of evidence, runtime checks, and scoped delivery.

## Requirements and validation scope

An actual conversion requires accessible H5 source, the target Unreal project, and the editing and build capabilities needed for the task. Use the Unreal Engine version and workflow established by the target project.

This repository contains skill instructions and reference documents. It does not include a sample Unreal project or a runtime plugin. Passing skill format checks and installing the skill successfully do not establish that a particular interface has been converted or has passed visual acceptance.

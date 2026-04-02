# Workspace Instructions

## Purpose

This workspace is for **generating PowerPoint presentations (.pptx files)**. All tasks, tools, and agent actions should be directed toward creating, editing, or managing PowerPoint decks.

## Templates

Use the `template-pptx` skill to resolve which corporate template to use. That skill manages all registered templates, their pre-extracted layouts, and onboarding new ones. **Do not hardcode template paths** — always go through the skill.

## Projects & Versioning

Each presentation project lives under `projects/`:

```
projects/<project_name>/<version>/
├── build.py              # build script (if programmatic)
├── source/               # content extracted from source documents
│   └── pages/            # individual page extracts
└── output/               # final .pptx deliverables only
```

- **Version folders** (`v1`, `v2`, …) track major iterations. Create a new version folder when the user requests a significant rework or a new take on the same project.
- **Never write output to the repo root or into the skill folders.**

## Workflow

1. Invoke the `template-pptx` skill to select and understand the template.
2. Use the `pptx` skill's editing workflow to build the presentation.
3. Save output to `projects/<project_name>/<version>/output/`.

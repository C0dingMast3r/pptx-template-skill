---
name: template-pptx
description: "Use this skill whenever creating or editing a PowerPoint presentation that should follow a corporate or branded template. Triggers include: any mention of 'template', 'corporate deck', 'branded presentation', or when AGENTS.md directs you to use a template. This skill manages available templates, their pre-extracted assets, and the workflow for onboarding new templates. Always invoke this skill BEFORE the pptx skill when a template is involved."
---

# Template PPTX Skill

Manages corporate/branded PowerPoint templates for this workspace. Templates live inside this skill at `templates/` so they're always available.

---

## Step 1 — Resolve the Template

Check what templates are available:

```
.agents/skills/template-pptx/templates/
```

List the subdirectories. Each subdirectory is one registered template.

| Scenario | Action |
|----------|--------|
| **No templates exist** | Ask the user to provide a `.pptx` template file. Then run the onboarding steps below. |
| **One template exists** | Use it automatically. |
| **Multiple templates exist** | Ask the user which template to use, listing the folder names. |

---

## Step 2 — Understand the Template

Each registered template folder has this structure:

```
templates/<template_name>/
├── <template_name>.pptx          # the original .pptx file
├── layout_inventory.txt          # layout index → name, placeholders, placeholder types
├── shape_inventory.txt           # per-slide shape details from the reference slides
├── images/                       # PNG screenshots of every slide (visual reference)
│   ├── Slide1.PNG
│   ├── Slide2.PNG
│   └── ...
└── unpacked/                     # full unzipped .pptx (XML, media, rels)
    ├── [Content_Types].xml
    ├── ppt/
    │   ├── presentation.xml
    │   ├── slideLayouts/
    │   ├── slideMasters/
    │   ├── slides/
    │   ├── theme/
    │   └── media/
    └── ...
```

Before building any presentation:

1. **Read `layout_inventory.txt`** to understand available layouts and their placeholder names/types.
2. **Read `shape_inventory.txt`** to understand what shapes exist on the reference slides.
3. **Browse `images/`** to visually understand each slide layout and design system.
4. **Note the fonts, color scheme, and branding** from the theme and master slides.

Use the `pptx` skill for the actual editing workflow (unpack → manipulate → pack), but use the pre-extracted assets here to skip the analysis step.

---

## Step 3 — Build the Presentation

Once you know which template to use, hand off to the `pptx` skill with these inputs:

- **Template .pptx path**: `templates/<template_name>/<file>.pptx`
- **Pre-extracted unpacked path**: `templates/<template_name>/unpacked/` (can copy from here instead of re-unpacking)
- **Layout reference**: from `layout_inventory.txt`

All paths above are relative to `.agents/skills/template-pptx/`.

### Output Location

Output goes to the workspace `projects/` folder, never into this skill's folder:

```
projects/<project_name>/<version>/
├── build.py                    # build script (if programmatic)
├── output/
│   └── <presentation>.pptx     # final deliverable
└── source/                     # extracted content from source docs
    └── ...
```

See AGENTS.md for the full project folder convention.

### Common Template Pitfalls

These issues come from not understanding what the layout already provides. **Before editing any slide, compare it against the layout in `layout_inventory.txt`** to see what's inherited.

- **Duplicate slide numbers / page numbers.** Layouts often include slide number placeholders. If you add another `<p:sp>` with a slide number, you get "12 12". Check the layout first — if it already has a footer or slide number placeholder, do not add one on the slide.
- **Misplaced slide numbers.** Slide number placeholders must stay in their layout-defined position (typically bottom-right). Never move them to the top-left or other arbitrary positions. If the layout already handles slide numbers, do not create a new text box with the number.
- **Duplicate footer / copyright text.** Same issue — layouts may already contain copyright notices or confidentiality text. Adding it again in the slide XML doubles it.
- **Content overflowing into footer zone.** The bottom ~0.75" of the slide is reserved for footer elements (copyright, page numbers). Content text boxes must not extend below `y + h > 6118225 EMU` (~6.69"). Check shape positions.
- **Media/image aspect ratio and letterboxing.** When inserting images or video placeholders, match the aspect ratio to the container. Black bars (letterboxing) indicate a mismatch. Size the container to the media's native aspect ratio, or crop to fit.
- **Broken or doubled decorative lines.** Layouts often have horizontal rule lines. If the slide XML also has one, you get visual artifacts (doubled, dashed, or misaligned lines). Remove the duplicate.
- **Title in wrong placeholder or position.** Always use the layout's designated title placeholder — do not create a free text box for the title. Check `layout_inventory.txt` for the correct placeholder idx and position.
- **Orphaned placeholder text.** After replacing content, grep for leftover template text like "Click to edit", "Your section title here", "Lorem ipsum", etc.

---

## Onboarding a New Template

When a user provides a new `.pptx` template:

1. **Create the template folder**:
   ```
   .agents/skills/template-pptx/templates/<template_name>/
   ```
   Use a lowercase, underscore-separated name derived from the file (e.g., `harman_corporate_2026`).

2. **Copy the .pptx** into that folder.

3. **Unpack** using the pptx skill's unpack script:
   ```bash
   python .agents/skills/pptx/scripts/office/unpack.py \
     ".agents/skills/template-pptx/templates/<template_name>/<file>.pptx" \
     ".agents/skills/template-pptx/templates/<template_name>/unpacked/"
   ```

4. **Generate slide thumbnails** for visual reference:
   ```bash
   python .agents/skills/pptx/scripts/thumbnail.py \
     ".agents/skills/template-pptx/templates/<template_name>/<file>.pptx"
   ```
   Move the generated images into `templates/<template_name>/images/`.

5. **Generate layout & shape inventories**:
   ```bash
   python .agents/skills/template-pptx/generate_inventories.py \
     ".agents/skills/template-pptx/templates/<template_name>/<file>.pptx"
   ```
   This creates `layout_inventory.txt` and `shape_inventory.txt` in the template folder.

6. **Confirm** to the user that the template is registered and ready.

---

## Notes

- **Never modify files inside `templates/`** during presentation generation. These are read-only reference assets.
- **Never delete or overwrite** a template without explicit user confirmation.
- The `unpacked/` folder is a convenience cache — it can always be regenerated from the `.pptx` file.

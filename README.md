# PPTX Builder

A 2-prompt setup that turns any corporate PowerPoint template into an AI-ready slide-building workspace.

## Requirements

- **GitHub Copilot**, **Codex**, or a similar AI coding agent
- Python 3.10+
- Node.js 18+

## Setup (2 Prompts)

### Prompt 1 — Initialize the workspace

```
Please have this folder be based on:
https://github.com/C0dingMast3r/pptx-template-skill
```

This pulls in the skills and folder structure needed for PPTX generation.

### Prompt 2 — Register your template

```
Please create a template for the following file
```

Attach your `.pptx` template file to the message. The agent will:

1. Extract and unpack the template
2. Generate slide thumbnails
3. Build layout and shape inventories
4. Register it as the active template

### Done

The agent is now ready. Just prompt with what you want to build:

- *"Create a 10-slide deck about Q3 results using the data in this PDF"*
- *"Build a pitch deck for a new product launch"*
- *"Add 3 slides to the existing presentation about market trends"*

The agent handles template selection, slide layout, content placement, and visual QA automatically.

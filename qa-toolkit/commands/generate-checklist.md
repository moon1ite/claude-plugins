---
description: "Generate QA checklist from a design source — dispatches specialized agents to verify design-to-implementation fidelity"
argument-hint: "[--feature <name>] [--source figma|html|screenshot|spec] [path-or-url]"
allowed-tools: ["Read", "Glob", "Grep", "Agent", "Write", "Bash"]
---

# QA Generate — Design Source → Checklist

> **Core question:** 기획의도 × 디자인의도가 구현에 충실하게 반영되어 있는가?

Generate a `qa-checklist.yaml` by dispatching specialized QA agents against a design source. Each agent verifies one dimension of design-to-implementation fidelity — not lint rules, but whether the product intent is faithfully built.

**Arguments:** "$ARGUMENTS"

## Supported Sources

| Source | How to provide | What agents extract from |
|--------|---------------|-------------------------|
| **Playground HTML** | File path or `--feature <name>` | CSS classes, JS state machine, rendered elements |
| **Figma** | Figma URL (`figma.com/design/...`) | Component tree, design tokens, layout specs via Figma MCP |
| **Screenshots** | Image file path(s) | Visual elements, text content, layout via vision |
| **Spec document** | Markdown file path | Requirements, user flows, expected behaviors |
| **Running app** | URL (with `--live`) | Actual rendered state via browser automation |

Agents adapt their extraction strategy based on the source type. The output format (YAML checklist) is always the same.

## Workflow

### Step 1: Resolve Source

Determine the source type and read it:

**Auto-detection:**
- `figma.com/` URL → Figma source (use Figma MCP `get_design_context`)
- `.html` file → Playground HTML
- `.png` / `.jpg` / `.webp` → Screenshot (use Read tool for vision)
- `.md` file → Spec document
- `--feature <name>` → look in `documents/product/{name}/playground/` for HTML
- `--live <url>` → Running app (use Playwright MCP for snapshot)

Read the source content into memory.

### Step 2: Discover Project Context

Discover what the project has — don't assume paths exist:

```
# Text constants (pick first match)
Glob: **/labels.ts, **/strings.ts, **/messages.ts, **/constants.ts
Glob: **/i18n/**, **/locales/**

# Design tokens (pick first match)
Glob: **/tokens/**, **/theme/**, **/design-tokens/**
Read: tailwind.config.* or nativewind config

# Writing rules (optional)
Glob: .claude/rules/*writing*, .claude/rules/*ux*
Read: CLAUDE.md for writing conventions

# Test users / seed data (optional)
Glob: **/seed*, **/dev-user*
Read: migration files for role/user IDs

# Existing checklist (for diffing)
Check: {feature_dir}/playground/qa-checklist.yaml
```

Pass discovered paths to each agent — agents adapt based on what exists.

### Step 3: Dispatch QA Agents in Parallel

Launch **all applicable agents simultaneously** using the Agent tool in a **single message**.

Pass each agent:
1. The source content (HTML, Figma context, screenshot, or spec text)
2. The source type (so agents adapt their extraction strategy)
3. Relevant project context files

#### screen-checker
```
subagent_type: "qa-toolkit:screen-checker"
```
- **From HTML**: parse render functions and sidebar controls
- **From Figma**: enumerate frames/pages and component variants
- **From screenshot**: identify visible screens and UI elements via vision
- **From spec**: extract screen descriptions and acceptance criteria

#### style-extractor
```
subagent_type: "qa-toolkit:style-extractor"
```
- **From HTML**: parse `<style>` block, extract CSS classes
- **From Figma**: extract spacing, radius, typography from design properties
- **From screenshot**: measure visual properties from image
- **From spec**: extract mentioned design values ("padding: 16px", "rounded corners")

#### ux-writing-checker
```
subagent_type: "qa-toolkit:ux-writing-checker"
```
- Extract all user-facing text from design source
- Compare against centralized text constants in the codebase
- Flag missing, mismatched, or hardcoded text

#### token-checker
```
subagent_type: "qa-toolkit:token-checker"
```
- **From HTML**: compare `:root` CSS vars vs discovered token files
- **From Figma**: compare Figma color styles vs tokens
- **From screenshot**: extract colors via vision, compare vs tokens
- **From spec**: skip (no token info in specs)

#### flow-tracer
```
subagent_type: "qa-toolkit:flow-tracer"
```
- **From HTML**: trace JS state machine functions
- **From Figma**: follow prototype connections between frames
- **From screenshot**: infer flow from sequential screenshots
- **From spec**: extract user stories and acceptance criteria flows

### Step 4: Consolidate into YAML

Merge all agent outputs into a single `qa-checklist.yaml`:

```yaml
meta:
  feature: {feature-name}
  source: {source-path-or-url}
  source_type: {html|figma|screenshot|spec|live}
  generated_at: {ISO date}

roles:
  - id: {role_name}
    user_id: "{uuid}"

checks:
  # ─── Screen Checks (from screen-checker) ──────
  ...

  # ─── Design Token Checks (from token-checker) ──
  ...

  # ─── Style Spec Checks (from style-extractor) ──
  ...

  # ─── UX Writing Checks (from ux-writing-checker)
  ...

  # ─── E2E Flow Checks (from flow-tracer) ────────
  ...
```

### Step 5: Diff Against Existing

If a `qa-checklist.yaml` already exists:
- Show what's new (added checks)
- Show what's removed (deleted checks)
- Show what's changed (modified values)
- Ask user to confirm before overwriting

### Step 6: Write and Commit

- Write `qa-checklist.yaml` to `{feature_dir}/playground/qa-checklist.yaml`
- Stage and commit: `docs(product): regenerate QA checklist from {source_type}`

## Report

After writing, print a summary:

```
# QA Checklist Generated

**Source:** {path-or-url} ({source_type})

| Agent | Checks |
|-------|--------|
| screen-checker | N screen checks |
| style-extractor | N style specs |
| ux-writing-checker | N writing rules |
| token-checker | N token comparisons |
| flow-tracer | N E2E flows |
| **Total** | **N checks** |

Saved to: {path}
```

## Usage Examples

**From playground HTML (default):**
```
/qa-generate --feature matches
```

**From Figma:**
```
/qa-generate https://figma.com/design/abc123/MyDesign
```

**From screenshots:**
```
/qa-generate docs/screenshots/contract-flow.png
```

**From spec:**
```
/qa-generate documents/frontend/0004_matches/01-requirements.md
```

## Tips

- **Run after playground updates**: regenerate checklist whenever the playground changes
- **Run before PRs**: catch design drift before it gets merged
- **Diff mode**: when checklist exists, shows what changed — review before overwriting
- **Source-specific agents**: not all agents apply to all sources (e.g., token-checker skips specs)

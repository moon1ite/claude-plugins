# qa-toolkit

QA checklist generator for Claude Code. Dispatches 5 specialized agents to verify that product intent (기획의도) and design intent (디자인의도) are faithfully implemented.

## Core Question

> 기획의도 × 디자인의도가 구현에 충실하게 반영되어 있는가?

## Agents

| Agent | What it verifies |
|-------|-----------------|
| **screen-checker** | All screens × roles × states from design are implemented |
| **style-extractor** | Visual properties (spacing, radius, typography) match design |
| **ux-writing-checker** | User-facing text matches design source copy |
| **token-checker** | Design tokens are correctly mapped and used |
| **flow-tracer** | User journeys and state transitions work end-to-end |

## Supported Sources

- **Playground HTML** — CSS classes, JS state machine, rendered elements
- **Figma** — Component tree, design tokens, layout specs
- **Screenshots** — Visual elements via vision analysis
- **Spec documents** — Requirements, user flows, acceptance criteria
- **Running app** — Live state via browser automation

## Usage

```
/qa-generate --feature matches
/qa-generate https://figma.com/design/abc123/MyDesign
/qa-generate docs/screenshots/contract-flow.png
```

## Output

Generates a `qa-checklist.yaml` with checks grouped by agent:

- Screen checks (text_visible, text_absent, element_count)
- Style specs (CSS property → NativeWind class mapping)
- UX writing (design text → code text fidelity)
- Design tokens (design colors → token values)
- E2E flows (multi-step user journey checks)

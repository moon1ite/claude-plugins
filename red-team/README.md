# red-team

Plan challenger toolkit for Claude Code. Dispatches 4 specialized agents to stress-test implementation plans before execution.

## Agents

| Agent | What it challenges |
|-------|-------------------|
| **risk-analyst** | Unstated assumptions and likely failure modes |
| **gap-analyzer** | Missing steps — error handling, rollback, validation, config |
| **dependency-challenger** | Task ordering — hidden dependencies, wrong sequencing, parallelism |
| **scope-challenger** | YAGNI violations, over-engineering, deferrable tasks |

## Usage

```
/challenge-plan
/challenge-plan docs/plans/2026-03-15-feature.md
```

## Output

Consolidated report with severity ratings and a verdict:
- **BLOCK** — critical issues, must fix before execution
- **REVISE** — high issues, should fix
- **PROCEED** — only medium or no issues

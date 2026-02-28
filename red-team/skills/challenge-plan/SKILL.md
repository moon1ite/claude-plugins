---
name: challenge-plan
description: Use after writing-plans to stress-test an implementation plan before execution. Dispatches red-team agents in parallel.
argument-hint: "[path-to-plan]"
allowed-tools: ["Read", "Glob", "Grep", "Task"]
---

# Red Team Challenge Plan

Stress-test an implementation plan by dispatching red-team agents in parallel, then consolidating findings into a single actionable report.

**Plan path (optional):** "$ARGUMENTS"

## Workflow

### Step 1: Locate the Plan

- If `"$ARGUMENTS"` is provided, read that file.
- Otherwise, use Glob to find the most recent `docs/plans/*.md` file (sorted by mtime). Read it.
- If no plan file found, tell the user and stop.

### Step 2: Read Context

- Read the plan file in full.
- Check for a `CLAUDE.md` at project root. If it exists, read it for project conventions.

### Step 3: Dispatch Red-Team Agents in Parallel

Launch **all four agents simultaneously** using the Task tool in a **single message**.

For each agent, pass the full plan text (and CLAUDE.md context if available) directly — don't make agents read files.

#### risk-analyst
```
subagent_type: "red-team:risk-analyst"
```
Ask it to surface unstated assumptions and likely failure modes.

#### gap-analyzer
```
subagent_type: "red-team:gap-analyzer"
```
Ask it to find missing steps — error handling, rollback, validation, config, security.

#### dependency-challenger
```
subagent_type: "red-team:dependency-challenger"
```
Ask it to analyze task ordering — hidden dependencies, wrong sequencing, parallelism opportunities.

#### scope-challenger
```
subagent_type: "red-team:scope-challenger"
```
Ask it to flag YAGNI violations, over-engineering, and tasks that can be deferred or cut.

### Step 4: Consolidate and Present

Merge all agent findings into one report. Classify by severity:

- **Critical** — Must fix before execution. Plan will likely fail without this.
- **High** — Should fix. Significant risk or debt.
- **Medium** — Consider fixing. Makes the plan more robust.

Use this format:

```markdown
# Red Team Report

**Plan:** [path] | **Agents:** 4 | **Verdict:** BLOCK / REVISE / PROCEED

## Critical (must fix)
- **[agent]**: [finding] → [suggested fix]

## High (should fix)
- **[agent]**: [finding] → [suggested fix]

## Medium (consider)
- **[agent]**: [finding] → [suggested fix]

## Strengths
- [what's well-designed]
```

**Verdict rules:**
- Any critical issues → **BLOCK**
- No critical but any high → **REVISE**
- Only medium or none → **PROCEED**

Then ask the user: revise the plan, re-run the challenge, proceed anyway, or drill into a specific agent's analysis.

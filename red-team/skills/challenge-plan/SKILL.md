---
name: challenge-plan
description: Use after writing-plans to stress-test an implementation plan before execution. Dispatches all challenger agents in parallel.
argument-hint: "[path-to-plan]"
allowed-tools: ["Read", "Glob", "Grep", "Task"]
---

# Red Team Challenge Plan

Stress-test an implementation plan by dispatching all red-team challenger agents in parallel, then consolidating their findings into a single actionable report.

**Plan path (optional):** "$ARGUMENTS"

## Workflow

### Step 1: Locate the Plan

- If `"$ARGUMENTS"` is provided and non-empty, use that as the path to the plan file. Read the file with the Read tool.
- If no argument is provided, use Glob to find the most recent `docs/plans/*.md` file. If multiple exist, pick the one with the most recent modification time (Glob results are sorted by mtime). Read that file.
- If no plan file is found, tell the user and stop.

### Step 2: Read Context

- Read the plan file content in full.
- Check if a `CLAUDE.md` file exists at the project root (use Glob for `CLAUDE.md`). If it exists, read it — this provides project-specific conventions, patterns, and constraints that agents need for accurate analysis.
- Store both the plan content and the CLAUDE.md content (if any) for passing to agents.

### Step 3: Dispatch All Red-Team Agents in Parallel

Launch **all five agents simultaneously** using the Task tool. Make all five Task tool calls in a **single message** so they run in parallel.

For each agent, provide:
- The full plan text (do NOT make agents read files — pass the content directly)
- The CLAUDE.md context if available
- A clear instruction to analyze the plan from their specific angle

**Launch these five agents in parallel:**

#### assumption-challenger
```
subagent_type: "red-team:assumption-challenger"
```
Prompt: Provide the full plan text and CLAUDE.md context. Ask it to identify all unstated assumptions — what does this plan take for granted about APIs, dependencies, environment, data, tooling, and team knowledge?

#### failure-premortem
```
subagent_type: "red-team:failure-premortem"
```
Prompt: Provide the full plan text and CLAUDE.md context. Ask it to assume the plan has already been executed and failed catastrophically, then work backward to identify the most likely root causes.

#### scope-challenger
```
subagent_type: "red-team:scope-challenger"
```
Prompt: Provide the full plan text and CLAUDE.md context. Ask it to flag YAGNI violations, over-engineering, unnecessary complexity, and tasks that could be deferred or removed for v1.

#### dependency-challenger
```
subagent_type: "red-team:dependency-challenger"
```
Prompt: Provide the full plan text and CLAUDE.md context. Ask it to analyze task dependencies and ordering — find hidden dependencies, incorrect sequencing, and missed parallelism opportunities.

#### gap-analyzer
```
subagent_type: "red-team:gap-analyzer"
```
Prompt: Provide the full plan text and CLAUDE.md context. Ask it to find what's MISSING from the plan — untested edge cases, unhandled error paths, missing rollback procedures, missing config changes, and missing documentation.

### Step 4: Consolidate Findings

After all five agents return their reports, merge their findings into a single structured document. Classify every issue by severity:

- **Critical** — Must address before execution. The plan will likely fail or cause serious damage if these are ignored.
- **High** — Should address before execution. Significant risk or technical debt.
- **Medium** — Consider addressing. Improvements that would make the plan more robust.

Organize the consolidated report using this exact format:

```markdown
# Red Team Challenge Report

**Plan reviewed:** [plan file path]
**Agents dispatched:** 5

## Critical Issues (must address before execution)
- **[agent-name]**: Issue description
  - Evidence: ...
  - Impact: ...
  - Suggested revision: ...

## High Priority (should address)
- **[agent-name]**: Issue description
  - Evidence: ...
  - Impact: ...
  - Suggested revision: ...

## Medium Priority (consider addressing)
- **[agent-name]**: Issue description
  - Evidence: ...
  - Impact: ...
  - Suggested revision: ...

## Suggested Plan Revisions (summary)
1. [revision from agent-name]
2. [revision from agent-name]
...

## Plan Strengths
- What the agents found well-designed

## Verdict
[BLOCK / REVISE / PROCEED]
- BLOCK: Critical issues found that would likely cause failure
- REVISE: High-priority issues that should be addressed before execution
- PROCEED: No critical or high issues, plan is ready for execution
```

**Verdict rules:**
- If ANY critical issues exist: **BLOCK**
- If no critical but any high-priority issues exist: **REVISE**
- If only medium or no issues: **PROCEED**

### Step 5: Present to User

Display the full consolidated report, then ask the user what they want to do next:
- Revise the plan based on the findings
- Re-run the challenge after revisions
- Proceed with execution despite the findings
- Drill deeper into a specific agent's analysis

## Agent Descriptions

**assumption-challenger:**
- Uncovers unstated assumptions about APIs, environment, data, and tooling
- Tests whether "it should work" claims have evidence
- Identifies compounding assumption risk

**failure-premortem:**
- Assumes the plan already failed and works backward
- Identifies most likely root causes of failure
- Focuses on boring, common failures over exotic edge cases

**scope-challenger:**
- Flags YAGNI violations and over-engineering
- Identifies tasks that can be deferred or removed
- Challenges premature abstractions and unnecessary complexity

**dependency-challenger:**
- Maps implicit dependency graphs between tasks
- Finds incorrect ordering and hidden dependencies
- Identifies parallelism opportunities to shorten the critical path

**gap-analyzer:**
- Finds what's NOT in the plan — missing steps, missing error handling
- Checks for rollback procedures, cleanup, and documentation
- Identifies missing tests and validation steps

## Usage Examples

**Challenge a specific plan:**
```
/red-team:challenge-plan docs/plans/add-authentication.md
```

**Challenge the most recent plan (auto-detect):**
```
/red-team:challenge-plan
```

**Typical workflow:**
```
1. Write your implementation plan to docs/plans/feature-name.md
2. Run: /red-team:challenge-plan docs/plans/feature-name.md
3. Review the consolidated report
4. Revise the plan based on critical and high-priority findings
5. Re-run: /red-team:challenge-plan docs/plans/feature-name.md
6. Once verdict is PROCEED, begin execution
```

## Tips

- **Run before execution, not after** — the whole point is to catch issues before you write code
- **Critical = stop** — if the verdict is BLOCK, do not proceed until critical issues are resolved
- **Agents are adversarial by design** — they will find issues even in good plans. Focus on critical and high severity.
- **Re-run after revisions** — a second pass often catches new issues introduced by fixes
- **Pass context** — if the plan references external docs or constraints, mention them in the plan file so agents can account for them

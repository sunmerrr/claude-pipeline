---
name: retrospective
description: Analyze pipeline artifacts and perform a retrospective. Reviews all stage outputs to identify patterns, agent quality issues, and opportunities for skill improvement.
---

# /retrospective - Pipeline Retrospective & Self-Improvement

You are a pipeline retrospective analyst. Your job is to review all artifacts from a pipeline run, identify patterns and issues, and optionally improve skill definitions to prevent recurring problems.

## Artifact Path

Determine where to read artifacts:
- If a pipeline workspace path is specified in the prompt (e.g., `.pipeline/{project-name}/`), use that path
- Otherwise, default to `.pipeline/` in the current working directory

## Step 1: Locate Artifacts

Check for the following artifacts in `{artifact-path}`:

| Artifact | Purpose |
|----------|---------|
| `research.md` | Research findings |
| `plan.md` | Implementation plan |
| `progress.md` | Implementation progress |
| `review.md` | Code review report |

```bash
ls -la {artifact-path}
```

- If no artifacts exist at all, tell the user:
  ```
  ⚠️ No pipeline artifacts found.
  Run /pipeline or individual skills first to generate artifacts.
  ```
  Then stop.

- If at least one artifact exists, proceed with whatever is available.

## Step 2: Read & Collect Data

Read all available artifacts and extract key information from each:

- **research.md**: approach chosen, libraries identified, constraints noted
- **plan.md**: task count, dependency structure, scope
- **progress.md**: completion status, any failed/skipped tasks
- **review.md**: issues found, severity distribution, build/test results

For any missing artifact, note it as "missing" — do not fail.

## Step 3: Analyze Patterns

Perform cross-artifact analysis looking for:

- **Recurring issues**: same type of bug appearing multiple times
- **Agent quality problems**: incomplete research, vague plans, missed review items
- **Process gaps**: missing artifacts, skipped steps
- **Scope accuracy**: plan vs actual implementation (plan.md vs progress.md)
- **Build/test health**: pass/fail trends from review.md

Categorize findings:

| Finding | Category | Severity | Affected Agent |
|---------|----------|----------|----------------|

Categories: `Process`, `Quality`, `Scope`, `Communication`
Severity: `High`, `Medium`, `Low`

Display the findings to the user before proceeding.

## Step 4: Skill Improvement (Optional)

If a pattern of problems is found that could be prevented by modifying a SKILL.md:

1. Identify which skill file to modify
2. Read the current SKILL.md content
3. Propose the specific change (new rule, clarification, additional step)
4. **Ask the user for confirmation before editing** — never auto-edit
5. If confirmed, make the minimal targeted edit
6. If no patterns warrant changes, note "No skill improvements needed"

## Step 5: Write Retrospective Report

Save to `{artifact-path}/retrospective.md`:

```markdown
# Retrospective: {project-name}

## Execution Summary
| Agent | Status | Notes |
|-------|--------|-------|
| Research | completed/skipped/missing | ... |
| Plan | completed/skipped/missing | ... |
| Implement | completed/partial/missing | ... |
| Review | completed/skipped/missing | ... |
| Bugfix | completed/skipped/missing | ... |

## Pipeline Health
- Build: pass/fail/unknown
- Tests: pass/fail/unknown
- Issues found in review: N (High: X, Medium: Y, Low: Z)

## Issues Found
- (problems encountered during the pipeline)

## Patterns & Observations
- (cross-cutting observations)

## Skill Improvements Made
- (list any SKILL.md modifications, or "None needed")

## Recommendations for Next Run
- (actionable suggestions)
```

Display summary to user:

```
━━━ Retrospective Complete ━━━
(artifact path)

(3-4 line summary)

→ Skill improvements: {Made N changes / None needed}
```

## Rules

- Read all available artifacts before analyzing — do not make assumptions about missing data
- If an artifact is missing, note it as "missing" in the summary rather than failing
- Do NOT commit changes — let the user decide when to commit
- Ask before modifying any SKILL.md files — never auto-edit without confirmation
- Skill edits should be minimal and targeted — add rules or clarifications, do not restructure
- Write in the same language the user has been using in the conversation

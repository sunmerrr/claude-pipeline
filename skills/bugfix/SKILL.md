---
name: bugfix
description: Fix issues found during code review. Reads .pipeline/review.md and fixes reported bugs, issues, and improvements.
---

# /bugfix - Review Issue Fixer

You are a debugging and code quality expert. Your job is to read the code review report and fix all identified issues.

## Artifact Path

Determine where to read artifacts:
- If a pipeline workspace path is specified in the prompt (e.g., `.pipeline/{project-name}/`), use that path
- Otherwise, default to `.pipeline/` in the current working directory

## Step 1: Read Review Report

Read `{artifact-path}/review.md`.

- If the file doesn't exist, tell the user:
  ```
  ⚠️ review.md not found.
  Run /review first to generate a code review report.
  ```
  Then stop.

- If the file exists, read it thoroughly and identify all actionable issues.

## Step 2: Parse Issues

Extract all issues from the review report. Look for:

- Items in the "주의사항" (Cautions/Warnings) section
- Items listed under test recommendations that indicate bugs
- Any explicitly called-out problems, warnings, or improvement suggestions
- Breaking changes that need fixes
- Missing error handling or edge cases

Categorize each issue:

| Priority | Type | Description |
|----------|------|-------------|
| 🔴 High | Bug / Breaking | Must fix — causes errors or breaks functionality |
| 🟡 Medium | Issue / Quality | Should fix — potential problems or code quality issues |
| 🟢 Low | Improvement | Nice to have — style, optimization, minor improvements |

Display the issue list to the user before starting fixes.

## Step 3: Fix Issues

Fix issues in priority order (High → Medium → Low). For each issue:

1. **Read the relevant file(s)** to understand the full context
2. **Identify the root cause** — don't just patch symptoms
3. **Apply the fix** following existing code patterns and conventions
4. **Verify the fix** doesn't break anything (run related tests if available)

Display progress for each fix:

```
━━━ Fix 1/N: (issue title) [진행 중] ━━━
(brief description of what you're fixing and how)
━━━ Fix 1/N: (issue title) [완료] ━━━
```

### Fix Guidelines

- Follow existing code patterns and conventions
- Make minimal, focused changes — don't refactor beyond what's needed
- If a fix requires significant changes, explain why before proceeding
- If you're unsure about a fix, ask the user rather than guessing

## Step 4: Summary

After all fixes are applied, display a summary:

```
━━━ Bugfix Complete ━━━

| # | Issue | Priority | Status |
|---|-------|----------|--------|
| 1 | (title) | 🔴 High | Fixed |
| 2 | (title) | 🟡 Medium | Fixed |
| 3 | (title) | 🟢 Low | Skipped — (reason) |

Files modified:
- path/to/file1.ts
- path/to/file2.ts
```

If there were no issues to fix, say so clearly:

```
━━━ Bugfix Complete ━━━
No actionable issues found in the review report. Code looks good! ✓
```

## Rules

- Read files before modifying them — understand existing code first
- Fix one issue at a time — don't batch unrelated changes
- Do NOT commit changes — let the user decide when to commit
- Do NOT introduce new features — only fix what was reported
- If a reported issue is actually not a problem (false positive), explain why and skip it
- Write in the same language the user has been using in the conversation

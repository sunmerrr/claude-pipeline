---
name: plan
description: Create an implementation plan with task breakdown based on research results. Reads .pipeline/research.md and produces a structured plan.
---

# /plan - Software Architect & Task Planner

You are a software architect. Your job is to take research findings and create a detailed, actionable implementation plan with properly ordered tasks.

## Artifact Path

Determine where to read/write artifacts:
- If a pipeline workspace path is specified in the prompt (e.g., `.pipeline/{project-name}/`), use that path
- Otherwise, default to `.pipeline/` in the current working directory

## Step 1: Read Research

Read `{artifact-path}/research.md`.

- If the file doesn't exist, tell the user:
  ```
  ⚠️ research.md not found.
  Run /research [feature description] first to explore the codebase.
  ```
  Then stop.

- If the file exists, read it thoroughly. Pay special attention to:
  - The recommended implementation approach
  - Related files and components
  - Existing patterns and conventions
  - Technical constraints

## Step 2: Design the Implementation

Based on the research:

1. **Identify all changes needed**: new files, modified files, new dependencies
2. **Group changes into logical tasks**: each task should be a coherent unit of work
3. **Define dependencies**: which tasks must complete before others can start
4. **Estimate scope**: each task should be small enough to implement in one focused session

### Task Design Guidelines

- Each task should modify 1-4 files (keep it focused)
- Tasks should be ordered so that foundational work comes first (models → services → routes → UI)
- Include any necessary configuration or setup as early tasks
- If tests are needed, include them as part of the relevant task or as a separate final task
- Avoid tasks that are too vague ("set up everything") or too granular ("add import statement")

## Step 3: Write the Plan

Write the plan to `{artifact-path}/plan.md` (create directory if needed):

```bash
mkdir -p {artifact-path}
```

The plan must follow the format defined in `~/.claude/skills/implement/references/plan-format.md`:

```markdown
# Plan: (overall goal — matches the research feature description)

(1-2 sentence overview of the implementation strategy, referencing the chosen approach from research)

## Task 1: (short task title)
- depends: none
- files: path/to/file1.ts, path/to/file2.ts

(Detailed description of what to do. Be specific:
- What to create or modify
- What patterns to follow (reference existing code)
- What the expected outcome is)

## Task 2: (short task title)
- depends: 1
- files: path/to/file3.ts

(Detailed description)

...
```

### Writing Good Task Descriptions

Each task description should include:
- **What** to create or change (specific files, functions, components)
- **How** to implement it (reference existing patterns from the research)
- **Why** this approach (brief rationale when not obvious)
- **Acceptance criteria** (how to know the task is done)

## Step 4: Summary

After writing the file, display:

```
━━━ Plan Complete ━━━
📄 {artifact-path}/plan.md

Tasks:
  1. (title) — (one-line summary)
  2. (title) — depends on 1
  3. (title) — depends on 1, 2
  ...

→ Next: run /implement to execute the plan
```

## Rules

- Do NOT modify any code — this is a planning-only step
- The plan must be compatible with the `/implement` skill's parser (use `## Task N:` format)
- Task numbers must be sequential starting from 1
- Every task must have a `depends:` line (use `none` if no dependencies)
- Every task must have a `files:` line listing affected files
- Be specific enough that each task can be implemented without ambiguity
- Write the plan in the same language the user has been using in the conversation
- Reference actual file paths and patterns discovered during research

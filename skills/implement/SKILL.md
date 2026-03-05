---
name: implement
description: Execute a multi-task implementation plan. Parses tasks with dependencies and executes them sequentially. Provide a plan file path as argument, or it will use the most recent file in ~/.claude/plans/.
disable-model-invocation: true
argument-hint: [plan-file-path]
---

# /implement - Multi-Task Plan Executor

You are executing a structured implementation plan. Follow these steps precisely.

## Step 1: Locate the Plan File

Determine the plan file location (check in this order):

1. If `$ARGUMENTS` is provided and non-empty → use that as the file path
   - If it's a relative path, resolve it relative to the current working directory
2. If a pipeline workspace path is specified in the prompt (e.g., `.pipeline/{project-name}/plan.md`), use that
3. If no argument → check `.pipeline/plan.md` in the current working directory
4. If not found → look for `.pipeline/*/plan.md` (any project subfolder)
5. If not found → find the most recently modified file in `~/.claude/plans/`
   ```bash
   ls -t ~/.claude/plans/ | head -1
   ```
6. If no plan file is found anywhere → tell the user and stop

Read the plan file and display its path to the user.

## Step 2: Parse Tasks

The plan file uses the format documented in `references/plan-format.md`. Parse all tasks from the file:

- Each task starts with `## Task N:` heading
- Extract: task number, title, dependencies, files, and description
- Build a dependency graph to determine execution order

Display a summary of all parsed tasks before starting execution.

## Step 3: Execute Tasks

Execute tasks sequentially respecting dependencies. Display progress as markdown text output:

```
━━━ Task 1/3: Create user model [진행 중] ━━━
```

For each task (in dependency order):

1. **Check dependencies**: All tasks listed in `depends:` must be completed first
2. **Print start banner**: `━━━ Task N/Total: Title [진행 중] ━━━`
3. **Execute the task**:
   - Read the relevant files listed in the task
   - Make the changes described in the task description
   - Verify the changes work (run tests if applicable)
4. **Print completion**: `━━━ Task N/Total: Title [완료] ━━━`
5. **Update progress file**: 태스크 완료 시마다 progress 파일을 갱신한다

### Progress Tracking

각 태스크가 완료될 때마다 `{artifact-path}/progress.md`를 **덮어쓰기**로 갱신한다. artifact-path는 프롬프트에서 지정된 `.pipeline/{project-name}/{task-slug}/` 경로를 사용하고, 없으면 `.pipeline/`을 사용한다.

```markdown
# Implement Progress

> Last updated: {YYYY-MM-DD HH:MM:SS}

## 진행 상황 ({completed}/{total})

  ✅ Task  1/{total}: {title}
  ✅ Task  2/{total}: {title}
  ⏳ Task  3/{total}: {title}  ← 진행 중
  ⬜ Task  4/{total}: {title}
  ...
```

이 파일은 매 태스크 완료 시마다 갱신되므로 외부에서 언제든 읽어서 진행 상황을 확인할 수 있다.

### On Task Failure

If a task fails:
- Keep it as `in_progress` (do not mark completed)
- Ask the user how to proceed:
  - **Retry**: Attempt the task again
  - **Skip**: Mark as completed and continue (may affect dependent tasks)
  - **Abort**: Stop execution entirely

### Dependency Resolution

- Tasks with `depends: none` or no depends line can run immediately
- A task is blocked until ALL its dependencies are completed
- If a circular dependency is detected, report it and stop

## Step 4: Completion

After all tasks are executed, print a summary table and run `/review`:

```
━━━ All 3 tasks done ━━━

| Task | Title              | Status |
|------|--------------------|--------|
| 1    | Create user model  | 완료   |
| 2    | Add auth middleware | 완료   |
| 3    | Add login endpoint | 완료   |
```

Then automatically execute the `/review` skill to summarize changes and recommend tests.

## Rules

- Execute tasks ONE AT A TIME in dependency order — do not skip ahead
- Read files before modifying them — understand existing code first
- Each task should be self-contained: make the changes, then verify
- Do NOT commit changes — let the user decide when to commit
- If the plan references files that don't exist, ask the user before creating them
- Communicate progress clearly throughout execution

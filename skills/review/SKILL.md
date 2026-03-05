---
name: review
description: Review code changes in the current branch or worktree. Summarizes what changed and why, then recommends testing approaches. Use when the user asks to review changes, summarize work, or wants test recommendations.
---

# /review - Code Change Review & Test Recommendations

You are reviewing the code changes in the current working directory.

## Step 1: Detect Base Branch

Try these branches in order to find the base:

```bash
git rev-parse --verify main 2>/dev/null    # try main first
git rev-parse --verify master 2>/dev/null   # then master
git rev-parse --verify develop 2>/dev/null  # then develop
```

Use the first one that exists as `<base>`. If none exist (e.g., initial commit or detached HEAD), adapt by comparing against the initial commit or just reviewing uncommitted changes.

## Step 2: Gather Change Information

Collect ALL of the following:

1. **Committed changes** (branch diff against base):
   ```bash
   git log --oneline <base>..HEAD
   git diff <base>...HEAD --stat
   git diff <base>...HEAD
   ```

2. **Uncommitted changes** (working directory):
   ```bash
   git diff --stat        # unstaged
   git diff --staged --stat  # staged
   git diff               # unstaged detail
   git diff --staged      # staged detail
   ```

Read the actual changed files to understand the full context of changes. Don't just rely on diffs — open and read the modified files when needed to understand what the code does.

## Step 3: Build & Test Verification

코드 리뷰 전에 **실제로 빌드와 테스트를 실행**해서 깨진 게 없는지 확인한다.

1. **프로젝트 타입 감지** — `package.json`, `Makefile`, `pyproject.toml`, `Cargo.toml` 등을 확인
2. **빌드 실행**:
   ```bash
   # JS/TS 프로젝트
   npm run build  # 또는 bun run build, pnpm run build 등

   # Python 프로젝트
   python -m py_compile <changed .py files>

   # 기타: 프로젝트에 맞는 빌드 명령 사용
   ```
3. **테스트 실행**:
   ```bash
   # JS/TS 프로젝트
   npm test  # 또는 bun test, pnpm test 등

   # Python 프로젝트
   python -m pytest

   # 기타: 프로젝트에 맞는 테스트 명령 사용
   ```
4. **결과 기록** — 빌드/테스트 결과를 리뷰 출력에 포함:
   - ✅ Build passed / ❌ Build failed (에러 내용 포함)
   - ✅ Tests passed (N개) / ❌ Tests failed (실패 목록 포함)
   - ⚠️ No test script found (테스트 명령이 없는 경우)

빌드나 테스트 스크립트가 없는 프로젝트는 해당 단계를 건너뛰고 리뷰 출력에 "빌드/테스트 스크립트 없음"으로 표시한다.

## Step 4: Produce Output

Output the review in the format defined in `references/output-format.md`. The review has 3 sections:

1. **Work Summary** — What was changed and why (2-5 bullets)
2. **Change Details** — Grouped by component/area, with notes on anything noteworthy
3. **Test Recommendations** — Automated tests, manual tests, and edge cases to check

## Step 4: Save Review Artifact

After producing the review output, save it for use by downstream pipeline stages.

Determine the artifact path:
- If a pipeline workspace path is specified in the prompt (e.g., `.pipeline/{project-name}/`), use that path
- Otherwise, default to `.pipeline/` in the current working directory

```bash
mkdir -p {artifact-path}
```

Write the full review output (the same content displayed to the user) to `{artifact-path}/review.md`.

If issues were found, add a note at the end of the output:

```
→ Issues found. Run /bugfix to fix them automatically.
```

## Rules

- Write the review in the same language the user has been using in the conversation (Korean if Korean, English if English, etc.)
- Be specific — mention actual file names, function names, and line numbers
- For test recommendations, prioritize practical tests the user can run immediately
- If there are no changes at all, say so clearly
- Do NOT make commits or modify any files — this is a read-only review (except writing .pipeline/review.md)

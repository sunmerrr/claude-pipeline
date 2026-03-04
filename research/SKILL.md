---
name: research
description: Research a feature via web search and codebase exploration. Investigates best practices, libraries, and APIs online, then analyzes the local codebase for patterns, dependencies, and constraints.
argument-hint: [feature description]
---

# /research - Web Researcher & Codebase Analyst

You are a senior developer doing research before implementation. Your job is to investigate the feature on the web (best practices, libraries, APIs) AND analyze the local codebase to produce a comprehensive research report.

## Step 1: Parse Input

Read the feature description from `$ARGUMENTS`.

- If `$ARGUMENTS` is empty or missing, ask the user what feature they want to research and stop.

### Artifact Path

Determine where to write artifacts:
- If a pipeline workspace path is specified in the prompt (e.g., `.pipeline/{project-name}/`), use that path
- Otherwise, default to `.pipeline/` in the current working directory

## Step 2: Web Research

Use WebSearch and WebFetch to investigate the feature. This step comes FIRST because it informs what to look for in the codebase.

### 2a. Technology Research

Search for:
- Best practices for implementing the feature (e.g., "best way to implement OAuth in Next.js 2025")
- Recommended libraries and tools (e.g., "best React form validation library comparison")
- Common architectural patterns for this type of feature
- Known pitfalls or gotchas

### 2b. Library & API Investigation

If relevant libraries or APIs are identified:
- Search for their official documentation
- Use WebFetch to read key doc pages (getting started, API reference)
- Check version compatibility, bundle size, maintenance status
- Compare popular alternatives (star count, community size, last update)

### 2c. Reference Implementations

Search for:
- Tutorials or guides that match the project's tech stack
- Open source examples of similar features
- Community discussions (Stack Overflow, GitHub issues) about common challenges

Collect the most useful findings with source URLs for the report.

## Step 3: Explore the Codebase

**먼저 프로젝트 존재 여부를 확인한다.** `package.json`, `tsconfig.json`, `requirements.txt`, `pyproject.toml`, `Cargo.toml`, `go.mod` 등 프로젝트 설정 파일이나 `src/`, `app/`, `lib/` 디렉토리가 하나도 없으면 → Step 3~5를 건너뛰고 Step 6으로 이동한다. 코드베이스 관련 섹션은 "신규 프로젝트 — 기존 코드 없음"으로 채운다.

**프로젝트가 존재하면**, 아래와 같이 탐색한다:

### 3a. Project Structure

- Use Glob to map the directory structure (`**/*` with depth limits)
- Identify the tech stack: check `package.json`, `requirements.txt`, `Cargo.toml`, `go.mod`, `build.gradle`, `Gemfile`, `pubspec.yaml`, or similar
- Identify the framework(s) in use (Next.js, Express, Django, Rails, etc.)
- Note the directory convention (e.g., `src/`, `app/`, `lib/`, `components/`)

### 3b. Related Code

- Use Grep to find code related to the feature description (search for keywords, similar features, related API endpoints, related components)
- Read the most relevant files to understand existing patterns
- Identify reusable utilities, shared components, or base classes

### 3c. Configuration & Dependencies

- Check relevant config files (tsconfig, eslint, webpack, vite, etc.)
- Note important dependencies and their versions
- Check for environment variables or secrets patterns (.env.example, etc.)

## Step 4: Analyze Patterns

From the code you've read, identify:

- **Coding style**: naming conventions, file organization, import patterns
- **Architecture patterns**: MVC, layered, feature-based, etc.
- **State management**: how data flows through the application
- **Error handling**: existing patterns for errors, validation
- **Testing patterns**: test framework, test file locations, test conventions

## Step 5: Synthesize & Assess

Combine web research findings with codebase analysis:

- Which libraries/approaches from web research are compatible with the existing stack?
- What existing code can be reused or extended?
- What new files/components need to be created?
- Are there technical constraints or compatibility issues?
- What dependencies (if any) need to be added?

## Step 6: Write Research Report

Create the artifact directory if it doesn't exist, then write the research report to `{artifact-path}/research.md`:

```bash
mkdir -p {artifact-path}
```

The report must follow this structure:

```markdown
# Research: (feature description)

## Web Research Findings

### Best Practices
- (key findings from web research about how to implement this feature)
- (recommended patterns and approaches from the community)

### Libraries & Tools
| Library | Purpose | Stars/Popularity | Compatibility | Notes |
|---------|---------|-----------------|---------------|-------|
| `lib-name` | What it does | Popularity indicator | Works with project? | Key pros/cons |

### Key References
- [Title](URL) — (one-line summary of what's useful)
- [Title](URL) — (one-line summary)

## Project Overview

- **Tech Stack**: (languages, frameworks, key libraries)
- **Architecture**: (pattern name + brief description)
- **Directory Structure**: (key directories and their purpose)

## Related Files & Components

| File | Role | Relevance |
|------|------|-----------|
| `path/to/file` | What it does | Why it's relevant |

## Existing Patterns

### Coding Style
- (naming conventions, formatting, etc.)

### Architecture Patterns
- (how similar features are structured)

### Key Conventions
- (anything important to follow)

## Technical Constraints & Dependencies

- (compatibility issues, version requirements, etc.)
- (external API limitations, if any)
- (performance considerations)

## Implementation Approaches

### Approach 1: (name) ⭐ Recommended
- **Description**: (how it works)
- **Based on**: (which web research findings support this)
- **Pros**: (advantages)
- **Cons**: (disadvantages)
- **Libraries needed**: (new dependencies, if any)
- **Effort**: (rough scope — small/medium/large)

### Approach 2: (name)
- **Description**: (how it works)
- **Based on**: (which web research findings support this)
- **Pros**: (advantages)
- **Cons**: (disadvantages)
- **Libraries needed**: (new dependencies, if any)
- **Effort**: (rough scope)

### Approach 3: (name) — optional
- (only if there's a meaningfully different third option)

## Recommendation

(1-2 paragraphs explaining why the recommended approach is best for this project, considering both web research findings and existing codebase patterns/constraints)
```

## Step 7: Summary

After writing the file, display a brief summary to the user:

```
━━━ Research Complete ━━━
📄 {artifact-path}/research.md

(3-4 line summary of findings and recommendation)

→ Next: run /plan to create an implementation plan
```

## Rules

- Do NOT modify any code — this is a read-only exploration
- Always do web research FIRST, then codebase exploration
- Be thorough but focused — explore what's relevant to the feature
- Always ground your analysis in actual sources (web URLs, actual code you've read), not assumptions
- Include source URLs for all web findings so the user can verify
- If the project is empty or has no code, note that and adjust the report accordingly (web research section should still be full)
- Write the report in the same language the user has been using in the conversation

# claude-pipeline

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
![Claude Code](https://img.shields.io/badge/Claude_Code-Skills-blueviolet)
[한국어](README.ko.md)

An AI development pipeline built on Claude Code custom skills.

Run `/pipeline` and a lead agent automatically dispatches specialized agents in sequence — from research through implementation, review, bugfix, and retrospective.

## How It Works

```
/pipeline add payment integration

→ Lead Agent (Pipeline) assesses the situation
  → Web Researcher (haiku)  — web search + information gathering
  → Code Analyzer (sonnet)  — codebase analysis + synthesis
  → Planner (opus)          — task decomposition + design
  → Implementer (sonnet)    — code implementation
  → Reviewer (opus)         — build/test + code review
  → Bugfixer (sonnet)       — issue resolution
  → Retrospective           — retrospective + skill self-improvement
```

Each agent runs in an isolated subprocess and communicates through markdown files in the `.pipeline/` directory.

## Agent Team

| Agent | Role | Model | Specialty |
|-------|------|-------|-----------|
| Web Researcher | Web search + info gathering | haiku | Fast, low-cost research |
| Code Analyzer | Codebase analysis + synthesis | sonnet | Pattern recognition, direction proposals |
| Planner | Task decomposition + design | opus | Architecture, deep reasoning |
| Implementer | Code implementation | sonnet | Code writing, speed/cost balance |
| Reviewer | Code review + build/test | opus | Thorough review, quality focus |
| Bugfixer | Issue resolution | sonnet | Targeted fixes, speed focus |

## Key Features

### Lead Agent's Autonomous Decisions

The Decision Matrix is a default guideline — the lead agent can skip or adjust steps based on the situation.

```
Priority: User Override > Autonomous Judgment > Decision Matrix (Default)
```

- Refactoring, internal logic changes → research can be skipped
- New tech adoption, external API integration → research required
- User says "skip research" → always respected

### Per-Task Workspace Isolation

Multiple tasks in the same project never collide — each gets its own artifact directory.

```
.pipeline/
  my-app/
    login-feature/
      research.md
      plan.md
      progress.md
      review.md
      retrospective.md
    payment-integration/
      research.md
      plan.md
      ...
```

### Build & Test Verification

The Reviewer doesn't just read diffs — it actually runs builds and tests.

### Implementation Progress Tracking

The Implementer updates `progress.md` after completing each task.

## Skills

| Skill | Description | Standalone |
|-------|-------------|------------|
| `/pipeline` | Full pipeline (lead agent dispatches as needed) | Yes |
| `/research` | Web research + codebase analysis | Yes |
| `/plan` | Create implementation plan | Yes |
| `/implement` | Plan-based implementation | Yes |
| `/review` | Code review + build/test | Yes |
| `/bugfix` | Fix issues from review | Yes |
| `/retrospective` | Pipeline retrospective + skill self-improvement | Yes |

Each skill can be used independently without `/pipeline`.

## Installation

Install directly in Claude Code:

```bash
/plugin marketplace add sunmerrr/claude-pipeline
/plugin install claude-pipeline@sunmerrr-claude-pipeline
```

### Recommended Permission Settings

To reduce permission prompts during pipeline execution, add to `~/.claude/settings.local.json`:

```json
{
  "permissions": {
    "allow": [
      "Bash(ls *)", "Bash(pwd)", "Bash(basename *)",
      "Bash(mkdir *)", "Bash(cat *)", "Bash(head *)", "Bash(tail *)",

      "Bash(git status*)", "Bash(git diff*)", "Bash(git log*)",
      "Bash(git branch*)", "Bash(git show*)", "Bash(git rev-parse*)",

      "Bash(npm run *)", "Bash(npm test*)", "Bash(npx *)",
      "Bash(bun run *)", "Bash(bun test*)", "Bash(bunx *)",

      "Bash(tsc *)", "Bash(eslint *)", "Bash(prettier *)",
      "Bash(python -m pytest*)", "Bash(python -c *)",

      "WebSearch"
    ]
  }
}
```

## Usage Examples

```bash
# Full pipeline
/pipeline add user authentication with OAuth2

# Skip research
/pipeline add notification icon to sidebar
# → Lead agent may autonomously skip research

# Individual skills
/research WebSocket real-time notifications
/plan
/implement
/review
/bugfix
```

## Artifact Flow

```
Web Researcher → web-research.md
                        ↓
Code Analyzer  → research.md (integrates web-research.md)
                        ↓
Planner        → plan.md (references research.md)
                        ↓
Implementer    → code implementation + progress.md
                        ↓
Reviewer       → review.md (includes build/test results)
                        ↓
Bugfixer       → code fixes (references review.md)
                        ↓
Retrospective  → retrospective.md
```

---

## Contributing

Contributions are welcome! Whether you want to fix a bug, improve an existing skill, or add a new one, please read [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

---

## License

MIT License - see [LICENSE](LICENSE) for details.

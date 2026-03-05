# Contributing to claude-pipeline

Thank you for your interest in contributing! claude-pipeline is a collection of Claude Code custom skills that automate AI-driven development workflows. All skill definitions are plain Markdown files — no build step required.

---

## How to Contribute

### Reporting bugs

1. Search [existing issues](https://github.com/sunmerrr/claude-pipeline/issues) to avoid duplicates.
2. Open a new issue using the **Bug Report** template.
3. Include the skill name, steps to reproduce, expected vs. actual behavior, and your environment.

### Requesting features

1. Search [existing issues](https://github.com/sunmerrr/claude-pipeline/issues) and [discussions](https://github.com/sunmerrr/claude-pipeline/discussions) first.
2. Open a new issue using the **Feature Request** template.
3. Describe the problem you are solving and the skill(s) it would affect.

### Improving existing skills

1. Fork the repository and create a branch (see [Pull Request Process](#pull-request-process)).
2. Edit the relevant `SKILL.md` file inside the skill directory (e.g., `skills/pipeline/SKILL.md`).
3. Test the change with Claude Code (see [Development Setup](#development-setup)).
4. Submit a pull request.

### Adding a new skill

1. Create a new directory under `skills/` with a descriptive name (e.g., `skills/deploy/`).
2. Add a `SKILL.md` file following the [Skill File Structure](#skill-file-structure).
3. Reference any supporting documents in a `references/` subdirectory if needed.
4. Update the skill table in `README.md`.
5. Submit a pull request.

---

## Development Setup

```bash
# 1. Fork the repository on GitHub, then clone your fork
git clone https://github.com/<your-username>/claude-pipeline.git
cd claude-pipeline

# 2. Install skills to ~/.claude/skills/
./install.sh

# 3. Open Claude Code in any project and test the modified skill
# Example:
/pipeline add a simple health-check endpoint
```

To re-install after editing:

```bash
./install.sh --force
```

---

## Skill File Structure

Each skill lives in its own directory:

```
skills/<skill-name>/
  SKILL.md          # The skill definition loaded by Claude Code
  references/       # (optional) Supporting context files referenced by SKILL.md
```

### SKILL.md conventions

- **Purpose block** — one-paragraph description of what the skill does and when to use it.
- **Instructions** — numbered or bulleted steps Claude should follow.
- **Output format** — describe the artifacts produced (e.g., file names, Markdown structure).
- **Examples** — concrete sample invocations.

Refer to the existing skills for examples:

- [`skills/pipeline/SKILL.md`](skills/pipeline/SKILL.md) — orchestrator pattern with sub-agent dispatch
- [`skills/research/SKILL.md`](skills/research/SKILL.md) — web search + codebase analysis
- [`skills/plan/SKILL.md`](skills/plan/SKILL.md) — structured task decomposition
- [`skills/implement/SKILL.md`](skills/implement/SKILL.md) — plan-driven implementation
- [`skills/review/SKILL.md`](skills/review/SKILL.md) — build, test, and code review
- [`skills/bugfix/SKILL.md`](skills/bugfix/SKILL.md) — targeted issue resolution

---

## Commit Message Convention

Format: `type(scope): description`

| Field | Values |
|-------|--------|
| `type` | `feat`, `fix`, `docs`, `refactor` |
| `scope` | Skill name: `pipeline`, `research`, `plan`, `implement`, `review`, `bugfix` |

**Examples:**

```
feat(research): add web search fallback strategy
fix(pipeline): correct agent dispatch order for refactor tasks
docs(implement): clarify progress.md update instructions
refactor(review): simplify build verification steps
```

---

## Pull Request Process

1. **Branch naming**

   | Prefix | Use for |
   |--------|---------|
   | `feat/` | New features or new skills |
   | `fix/` | Bug fixes |
   | `docs/` | Documentation-only changes |

   Example: `feat/deploy-skill`, `fix/pipeline-dispatch`

2. **Fill out the PR template** — summary, related issue, affected skills, and checklist.

3. **Review process**
   - A maintainer will review your PR, typically within a few days.
   - Address any requested changes in follow-up commits on the same branch.
   - Once approved, a maintainer will merge the PR.

---

## Code of Conduct

This project follows the [Contributor Covenant v2.1](https://www.contributor-covenant.org/version/2/1/code_of_conduct/). Please be respectful and constructive in all interactions.

---

## License

By contributing, you agree that your contributions will be licensed under the [MIT License](LICENSE).

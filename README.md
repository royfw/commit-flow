# commit-flow

AI-assisted commit workflow conventions for consistent, traceable commits.

## What It Covers

1. **Changeset creation** — Write `.changeset/*.md` files directly (no interactive CLI). Supports single package and monorepo
2. **Conventional Commits** — `type(scope): description` format with paragraph or bullet list body
3. **Co-Authored-By signature** — Dynamically reads model name from `ANTHROPIC_MODEL` / `CLAUDE_MODEL` / `MODEL_NAME` env vars
4. **Commit granularity** — Split by change nature, not file count
5. **Pre-commit checklist** — `git status` → security scan → `git diff` → `git log` → draft message → user confirmation
6. **Branch decisions** — When to create a new branch vs commit on current branch
7. **Security check** — Reject `.env`, `*.key`, `*.pem`, credentials, secrets before staging
8. **Release flow** — `npx @changesets/cli version` → commit → tag

## Changeset Body Format

Single change — paragraph:

```markdown
---
"package-name": minor
---

feat(scope): short title

Detailed paragraph explaining what changed and why. This appears in CHANGELOG.md.
```

Multiple changes — bullet list:

```markdown
---
"@myorg/core": minor
"@myorg/cli": patch
---

feat(core): add connection pooling

- Add pool size configuration via POOL_SIZE env var
- Implement idle connection timeout (default 30s)
- Update CLI to pass pool config to core library
```

## Installation

Copy or symlink `SKILL.md` to your skills directory:

```bash
mkdir -p ~/.claude/skills/commit-flow
ln -s "$(pwd)/SKILL.md" ~/.claude/skills/commit-flow/SKILL.md
```

Restart Claude Code for the skill to take effect.

> **Tip:** Using a symlink (`ln -s`) means updates to this repo automatically apply — no need to re-copy.

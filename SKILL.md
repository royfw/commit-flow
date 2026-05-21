---
name: commit-flow
description: Commit workflow conventions: changeset creation, Conventional Commits formatting, and Co-Authored-By signing. Applies when creating commits, writing commit messages, making changes that should be tracked, or running releases.
metadata:
  version: "1.0"
---

## Commit Flow

This skill defines the complete commit workflow for AI-assisted development.

---

### 1. Changeset (if the repo has `.changeset/`)

When making meaningful changes in a repo that has `.changeset/config.json`, create a changeset file before committing.

**If the repo does NOT have `.changeset/`**: skip this section and proceed to the commit message and pre-commit checklist. If the repo has multiple packages or no changelog process, suggest setting it up:

> This repo doesn't use changesets. Would you like to set up `@changesets/cli` for automated changelog generation?

**Create the file directly** — do not use interactive CLI:

```
.changeset/{adjective}-{noun}-{id}.md
```

#### Single Package Repo

```markdown
---
"deploy-toolkit": patch
---

feat(scope): short title

Detailed description of what changed and why. This paragraph appears in CHANGELOG.md.
```

#### Monorepo

List each affected package with its bump type:

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

**Bump types (semver mapping):**
- `patch` — bug fixes, docs updates, minor tweaks. No new functionality, no breaking changes. Corresponds to `fix:`, `docs:`, `chore:`, `style:`
- `minor` — new features or improvements that are backward-compatible. Corresponds to `feat:`
- `major` — breaking changes that require consumers to update their code or configuration. Corresponds to `feat!:` or `fix!:`
- `npx @changesets/cli --empty` — changes that don't need a version bump at all

**When in doubt between patch and minor:** if a consumer would need to know about the change (new behavior, new option, new endpoint), it's minor. If they wouldn't notice, it's patch.

**When in doubt, ask the user** which bump type fits — especially for changes that could be interpreted either way.

**Body convention:** Use Conventional Commits prefix (`feat:`, `fix:`, `docs:`, `chore:`) with scope in parentheses on the first line. Add a blank line, then a detailed paragraph or bullet list when there are multiple discrete changes. This matches the CHANGELOG format.

---

### 2. Commit Message

Use Conventional Commits format: `type(scope): description`

**Types:** `feat`, `fix`, `docs`, `chore`, `refactor`, `test`, `style`, `perf`

**Co-Authored-By Signature**

Every git commit MUST include a `Co-Authored-By` trailer line.

**Model name resolution** — BEFORE presenting the commit summary, run a shell command to check the actual values:

```bash
echo "ANTHROPIC_MODEL=${ANTHROPIC_MODEL:-<empty>}"
echo "CLAUDE_MODEL=${CLAUDE_MODEL:-<empty>}"
echo "MODEL_NAME=${MODEL_NAME:-<empty>}"
```

Use the first non-empty value from:
1. `ANTHROPIC_MODEL` env var
2. `CLAUDE_MODEL` env var
3. `MODEL_NAME` env var

If none are available, omit the trailer. Do NOT guess or assume the values — always read them from the shell.

**Format:**

```
feat(scope): short description

<blank line>

Body: a paragraph explaining the "why", or a bullet list when there are
multiple discrete changes. Keep each bullet concise.

Co-Authored-By: <model-name> <noreply@anthropic.com>
```

**Body styles:**

Single change — use a paragraph:

```
feat(chart): add nodeSelector and affinity support

Allow consumers to schedule pods to specific nodes via Helm overlay values.

Co-Authored-By: qwen3.6-27b-code-act <noreply@anthropic.com>
```

Multiple changes — use a bullet list:

```
feat(chart): add scheduling and autoscaling support

- Add nodeSelector, tolerations, and affinity to deployment and job templates
- Add HPA (HorizontalPodAutoscaler) template for autoscaling
- Remove default resource fallback so overlays control resources entirely

Co-Authored-By: qwen3.6-27b-code-act <noreply@anthropic.com>
```

**Important:**
- Read the env variable at commit time — do not hardcode any model name.
- When using heredoc, use unquoted `EOF` (not `'EOF'`) so the shell expands the variable.

**Example:**

```bash
git commit -m "$(cat <<EOF
feat(chart): add nodeSelector and affinity support

Allow consumers to schedule pods to specific nodes via Helm overlay values.

Co-Authored-By: ${ANTHROPIC_MODEL} <noreply@anthropic.com>
EOF
)"
```

---

### 3. Release Flow (for repos using changesets)

```bash
# 1. Bump version and update CHANGELOG.md
npx @changesets/cli version

# 2. Review changes, then commit and tag
NEW_VERSION=$(jq -r '.version' package.json)
git add package.json CHANGELOG.md .changeset/
git commit -m "$(cat <<EOF
chore(release): bump version to v${NEW_VERSION}

Co-Authored-By: ${ANTHROPIC_MODEL} <noreply@anthropic.com>
EOF
)"
git tag v${NEW_VERSION}
```

---

### 4. Commit Granularity

Split commits by **change nature**, not by file count:

- **Separate commits** for independent features, fixes, or unrelated changes
- **One commit** for changes that belong together (e.g., code + its docs, a script + its test)
- **Changeset files** go in the same commit as the changes they describe — never a standalone "add changeset" commit

**Examples:**
- Fixed a bug in `deploy-service.sh` AND added a new Helm template → 2 commits
- Updated `deploy-service.sh` AND the docs describing it → 1 commit
- Three typo fixes across different files → 1 commit

### 5. Pre-Commit Checklist

Before committing, review in this order:

1. **`git status`** — confirm all intended files are present, no surprises
2. **Security check** — scan for `.env`, `*.key`, `*.pem`, `credentials.json`, `id_rsa`, secrets files. Exclude if found.
3. **`git diff`** (unstaged) + **`git diff --cached`** (staged) — verify changes are correct
4. **`git log --oneline -5`** — check recent commit message style for consistency
5. **Check model env vars** — run `echo "ANTHROPIC_MODEL=${ANTHROPIC_MODEL:-<empty>}"` etc. to resolve Co-Authored-By name. Do NOT guess.
6. **Draft the commit message** based on what you observed, present to user for confirmation
7. **User confirms** → stage specific files → commit

Present a summary before committing:

- Files to be staged
- Proposed commit message (subject + body)
- If changeset applies: proposed bump type (`patch`/`minor`/`major`) and changeset body

Wait for user confirmation before executing `git add` and `git commit`.

### 6. Branch Decisions

**Always create the branch before committing.** Uncommitted changes are carried over automatically with `git checkout -b`, so there's no need to commit first and then move commits.

- **Feature work, refactors, multi-commit tasks** → `git checkout -b feat/short-description`
- **Bug fixes** → `git checkout -b fix/short-description`
- **Small fixes on main** (typo, one-line fix, docs tweak) → commit directly on current branch
- **User is already on a feature branch** → commit there, no need to create another

When in doubt, ask the user rather than assuming.

### 7. Security Check

Before staging files, scan the list for sensitive content. Never commit:

- `.env`, `*.env`, `*.env.*` files
- `*.key`, `*.pem`, `*.p12`, `*.pfx`
- `credentials.json`, `service-account*.json`
- `id_rsa`, `id_ed25519` (without `.pub`)
- `secrets.*`, `secret.*` (unless in a clearly intended secrets directory)
- Any file containing API keys, tokens, passwords, or private keys

If you notice such files in the staging area, warn the user and exclude them.

### 8. General Rules

- Create NEW commits rather than amending, unless explicitly asked.
- Stage specific files by name — never `git add .` or `git add -A` (may include sensitive files).
- Commit message subject describes the "why", body provides context only when non-obvious.
- Do not commit unless explicitly asked by the user.

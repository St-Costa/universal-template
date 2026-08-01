# Using this template

Checklist for turning this template into a real project. Delete this file once done.

## 1. Replace placeholders

Search the repo for these tokens and replace them:

| Token | Replace with |
|---|---|
| `{{PROJECT_NAME}}` | Project name |
| `{{PROJECT_DESCRIPTION}}` | One-paragraph description (in `CLAUDE.md`) |
| `{{OWNER}}` | GitHub org/user |
| `{{REPO}}` | Repo name |
| `{{AUTHOR}}` | Copyright holder (in `LICENSE`) |
| `{{YEAR}}` | Current year (in `LICENSE`) |
| `{{SECURITY_CONTACT_EMAIL}}` | Where security reports should go (in `SECURITY.md`) |

```bash
# example: find every placeholder token left to fill in
grep -rn '{{[A-Z_]*}}' --include='*.md' --include='*.yml' .
```

## 2. Pick your stack

This template intentionally ships no language-specific tooling. For your stack:

- Add real dependency/build files (`package.json`, `pyproject.toml`, `go.mod`, `Cargo.toml`, etc.)
- Replace the placeholder job in [.github/workflows/ci.yml](../.github/workflows/ci.yml) with real install/lint/test/build steps.
- Fill in the `Commands` table in [CLAUDE.md](../CLAUDE.md) and the `Getting started` section in [README.md](../README.md).
- Uncomment the relevant section of [.gitignore](../.gitignore).

## 3. Fill in CLAUDE.md

This is the single most valuable file for AI-assisted development. At minimum, fill in:

- Project overview
- Tech stack
- Commands table
- Git & PR conventions

See [docs/ai/maintaining-claude-md.md](ai/maintaining-claude-md.md) for how to keep it accurate over time.

## 4. Decide on CODEOWNERS

Edit [.github/CODEOWNERS](../.github/CODEOWNERS) or delete it if not needed yet.

## 5. Clean up

- Delete this file.
- Delete `src/.gitkeep` and `tests/.gitkeep` once real files exist there.
- Update the README badge URLs once the repo exists on GitHub with CI configured.
- If you don't need ADRs, delete `docs/adr/` — but consider keeping [ADR 0001](adr/0001-record-architecture-decisions.md) since it explains why the practice exists.

## 6. First commit

Reset history so the new project doesn't inherit template commits (optional, but common):

```bash
rm -rf .git
git init
git add .
git commit -m "chore: initialize project from template"
```

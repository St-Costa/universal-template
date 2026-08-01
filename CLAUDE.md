# CLAUDE.md

Instructions for Claude Code (and other AI coding agents that read this file) when working in this repository.

## Project overview

<!-- Replace with a short, concrete description: what this project is, who uses it, and what "done" looks like for a typical change. -->

{{PROJECT_DESCRIPTION}}

## Tech stack

<!-- List language(s), frameworks, package manager, and any non-obvious infra (database, queue, deploy target). Keep this current — it's the first thing an agent reads. -->

- Language:
- Package manager:
- Test framework:
- Lint/format tooling:
- Deploy target:

## Commands

<!-- Exact, copy-pasteable commands. Agents will run these verbatim. -->

| Task | Command |
|---|---|
| Install deps | `` |
| Run dev server | `` |
| Run tests | `` |
| Run a single test | `` |
| Lint | `` |
| Format | `` |
| Build | `` |
| Type-check | `` |

## Conventions

- Follow the style already present in the file you're editing over any external style guide.
- Prefer editing existing files over creating new ones.
- No speculative abstractions: build what's asked, not what might be needed later.
- Keep functions/files close to the size and shape of neighboring code in the same directory.
- Comment only where the *why* isn't obvious from the code itself.

<!-- Add project-specific conventions: naming, error handling patterns, where business logic lives vs. handlers, etc. -->

## Testing expectations

- New behavior needs a test. Bug fixes need a regression test.
- Run the full test suite before considering a task done; do not just run the one test you wrote.
- Do not weaken or delete a test to make it pass.

## Git & PR conventions

- Commit messages: <!-- e.g. Conventional Commits (`feat:`, `fix:`, `chore:`) or plain imperative subject lines -->
- Branch naming: <!-- e.g. `feature/short-description`, `fix/short-description` -->
- Keep PRs scoped to one logical change. Explain *why* in the PR description, not just *what*.
- Do not commit secrets, credentials, or `.env` files. See `.gitignore`.

## What not to touch without asking

<!-- List anything sensitive: migrations, CI/CD config, infra-as-code, generated files, vendored code. -->

- CI/CD workflow files (`.github/workflows/`) — changes here affect every future PR.
- Anything under `docs/adr/` — these are historical records, not living docs; add a new ADR instead of editing an old one.

## Architecture notes

<!-- Point to docs/adr/ for detailed decisions. Use this section only for a short map of major components and how they talk to each other. -->

See [docs/adr/](docs/adr/) for the reasoning behind major architectural decisions.

## Keeping this file useful

This file should describe the *current* state of the project, not its history. When you make a change that invalidates something above (a new command, a changed convention, a new "don't touch" area), update this file in the same PR. See [docs/ai/maintaining-claude-md.md](docs/ai/maintaining-claude-md.md) for guidance.

# CI/CD pipeline security

This covers the security of the pipeline itself — the YAML/config that builds, tests, and deploys the project — as distinct from [CLAUDE.md § Dependency & supply-chain security](../../CLAUDE.md#dependency--supply-chain-security), which covers dependencies *inside* the application. Both are supply-chain concerns; this one is about who/what can act on the repo and its secrets, not about vulnerable libraries in the app.

Applies regardless of language/stack, since it's about the pipeline config, not the code it builds. Written for GitHub Actions since that's what this template ships (`.github/workflows/ci.yml`); the same principles carry over to other CI systems (GitLab CI, CircleCI, etc.) with different syntax.

Changes to `.github/workflows/` require user confirmation per [CLAUDE.md § What not to touch without asking](../../CLAUDE.md#what-not-to-touch-without-asking) — this doc is what to check *when* reviewing or proposing such a change, not a license to make it unasked.

## Checklist

- [ ] **Actions pinned to a full commit SHA**, not a mutable tag (`uses: actions/checkout@<sha>  # v4.x.x`, not `@v4` or `@main`). A tag can be moved by the action's maintainer (or an attacker who compromises their account) to point at malicious code without any change visible in this repo's history. Keep the version comment for readability; Dependabot/Renovate can keep the pin current if configured.
- [ ] **Minimal `permissions:`** set explicitly at the workflow or job level (`contents: read` unless a job actually needs to write — e.g. publishing a release, commenting on a PR). Don't rely on the default token scope, which is broader than most jobs need.
- [ ] **No secrets in logs**: secrets aren't echoed, printed in debug output, or passed as plain CLI arguments that show up in process listings/logs. Use the platform's secret-masking (`secrets.X` via env var), not string interpolation into a shell command.
- [ ] **`pull_request_target` / secrets on fork PRs treated with care**: workflows triggered by PRs from forks should not run untrusted code (checked-out PR branch) with access to repo secrets. If a workflow needs both (e.g. to post a comment), split into a trusted job with no checkout of PR code and an untrusted job with no secret access.
- [ ] **Third-party actions scrutinized like any other dependency** before adding: is it maintained, reasonably popular, from a known publisher? Prefer official/verified actions (`actions/*`, platform-provided) over an obscure third-party one that does something trivial you could inline.
- [ ] **No `curl | sh` / arbitrary remote script execution** in a workflow step without pinning the fetched content (checksum or pinned ref) — same reasoning as the app-dependency rule against piping remote scripts into a shell.
- [ ] **Branch protection / required status checks** (repo settings, not workflow YAML, but worth confirming): the checks that matter (tests, lint, dependency audit) are marked required so they can't be silently skipped on a PR.
- [ ] **Deploy credentials scoped narrowly**: a workflow that deploys uses a token/role scoped to only what that deploy needs (e.g. a specific cloud project/environment), not a broad admin credential reused across jobs.

## When adapting the template

When wiring up the real CI job in [.github/workflows/ci.yml](../../.github/workflows/ci.yml) (see [docs/USING_THIS_TEMPLATE.md](../USING_THIS_TEMPLATE.md)):

- Include the dependency audit command (`make audit`) as a required step, not an afterthought.
- Pin every `uses:` action added, by SHA, from the start — retrofitting pins later means auditing history you didn't write.
- Set `permissions: contents: read` at the workflow level as the default, then widen per-job only where needed.

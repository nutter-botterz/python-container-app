# Agent Commit Guide

Quick reference for how Jarvis commits code.

## When to Commit

- After completing a logical unit of work
- Before merging to main
- When tests pass

## How to Commit

### Single change:
```bash
git add <files>
git commit -m "type(scope): description"
```

### Interactive staging:
```bash
git add -p  # review hunks interactively
git commit
```

## Commit Types

| Type | Use for |
|------|---------|
| `feat` | New functionality |
| `fix` | Bug fixes |
| `docs` | README, comments, docs |
| `style` | Formatting only |
| `refactor` | Restructuring code |
| `test` | Adding tests |
| `chore` | Deps, build, CI |

## Tips

- Write imperative mood ("add" not "added")
- Keep subject line under 50 chars
- Body wrapped at 72 chars
- Reference issues: `Closes #123`

## Example Workflow

```bash
# Make changes to files
git status
git add -A
git commit -m "feat(auth): implement login flow

- Add user model
- Create login endpoint
- Add session handling

Closes #45"
git push
```

---

# Release Process

This project uses **semantic-release** with GitHub Actions for automated releases.

## How Releases Work

1. **Push to `main`** — Triggers a full release (production)
2. **Push to prerelease branches** — `next`, `beta`, `alpha` — creates prereleases
3. **Semantic-release** analyzes commits since last release to determine version bump
4. **Docker image** is built and pushed to GHCR with the new tag

## Branch Strategy

| Branch | Release Type |
|--------|--------------|
| `main` | Production release |
| `next` | Prerelease (e.g., `1.0.0-next.1`) |
| `beta` | Prerelease (e.g., `1.0.0-beta.1`) |
| `alpha` | Prerelease (e.g., `1.0.0-alpha.1`) |

## Version Bumping

Semantic-release automatically determines the version bump based on commit messages:

| Commit Message | Version Bump |
|----------------|--------------|
| `fix:` | Patch (`1.0.0` → `1.0.1`) |
| `feat:` | Minor (`1.0.0` → `1.1.0`) |
| `feat!:`, `fix!:`, etc. | Major (`1.0.0` → `2.0.0`) |

## Triggering a Release

```bash
# For production release (main branch)
git checkout main
git merge your-feature-branch
git push origin main

# For prerelease (beta branch)
git checkout beta
git merge your-feature-branch
git push origin beta
```

## Release Outputs

- **GitHub Release** — Created automatically with changelog (title is version only, e.g., `v1.2.0`)
- **Docker Image** — Pushed to container registry
  - GHCR: `ghcr.io/<owner>/<repo>:<tag>`
  - ECR: `<account>.dkr.ecr.<region>.amazonaws.com/<repo>:<tag>`
  - Prerelease: `pre-<branchname>` (e.g., `pre-beta`)

## Important Notes

- **Don't specify versions** in commit messages — semantic-release determines the version
- All conventional commits are collected since the last release
- GitHub release title is just the version number (e.g., `v1.2.0`), no description

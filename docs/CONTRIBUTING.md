# Contributing

Welcome! This guide covers how to contribute to this project.

## Commit Format

We follow [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>(<scope>): <subject>

[optional body]
```

### Types

| Type | Description |
|------|-------------|
| `feat` | New feature |
| `fix` | Bug fix |
| `docs` | Documentation |
| `style` | Formatting |
| `refactor` | Code restructuring |
| `test` | Tests |
| `chore` | Maintenance |

### Examples

```bash
git commit -m "feat(api): add health check endpoint"
git commit -m "fix(docker): correct port mapping"
git commit -m "docs(readme): update installation steps"
```

### Breaking Changes

```bash
git commit -m "feat(api)!: change response format
Closes #123"
```

## Branch Strategy

- **`main`** — Stable release branch
- **`feat/*`** — Feature branches (create PR to main)
- **`fix/*`** — Bug fix branches

## Pre-releases

Pushing to any branch other than `main` triggers a canary build:
- Image tagged as `ghcr.io/<owner>/<repo>:<branch-name>`
- Pre-release on GitHub

## Releases

Merging to `main` triggers release-please:
- Automatically bumps version based on commits
- Creates changelog
- Builds and pushes release image as `ghcr.io/<owner>/<repo>:<version>`

## Pull Requests

1. Create a feature or fix branch
2. Make your changes
3. Commit using conventional format
4. Push and open a PR
5. Await review

## Questions?

Open an issue for discussion before starting major work.

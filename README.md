# Python Container

A simple Flask container for testing CI/CD with GitHub Actions.

## Quick Start

### Run locally

```bash
pip install -r infra/container/requirements.txt
python src/app.py
```

### Run with Docker

```bash
docker build -t python-container -f infra/container/Dockerfile .
docker run -p 5000:5000 python-container
```

## Architecture

- **Container Registry**: Amazon ECR
- **Compute**: AWS ECS Fargate
- **CI/CD**: GitHub Actions with OIDC

### Tags

| Tag | Description |
|-----|-------------|
| `latest` | Latest stable release (main branch) |
| `1.0` | Major.minor release |
| `1` | Major release |
| `branchname` | Prerelease/canary build from branch |

## AWS Setup

To deploy to ECS, you need to configure AWS OIDC:

See [docs/AWS_OIDC_SETUP.md](docs/AWS_OIDC_SETUP.md) for detailed instructions on:
- Creating IAM OIDC identity provider
- Creating IAM role with restricted access
- Configuring GitHub secrets and variables

## Development

See [CONTRIBUTING.md](CONTRIBUTING.md) for commit conventions and workflow.

## For Agents

After making changes, update the following files:

1. **README.md** — Update usage docs if you add/change features
2. **agent.md** — Update commit/release instructions if the process changes
3. **infra/aws/task.json** — Update ECS task definition if needed

### Commit Process

See [agent.md](agent.md) for detailed instructions on:
- How to commit code
- How to trigger releases
- Branch strategy

# AWS OIDC Setup Guide

This guide covers how to configure AWS for GitHub Actions OIDC authentication.

## Overview

We use AWS IAM OIDC Identity Provider to allow GitHub Actions to assume a role without storing long-lived credentials.

## Prerequisites

- AWS CLI installed locally
- Sufficient IAM permissions to create roles and identity providers

---

## Step 1: Create OIDC Identity Provider in IAM

### Option A: Using AWS Console

1. Navigate to **IAM → Identity providers → Add provider**
2. Select **OpenID Connect**
3. Provider URL: `https://token.actions.githubusercontent.com`
4. Audience: `sts.amazonaws.com`
5. Click "Add provider"

### Option B: Using AWS CLI

```bash
aws iam create-open-id-connect-provider \
  --url https://token.actions.githubusercontent.com \
  --client-id-list sts.amazonaws.com \
  --thumbprint-list 6938FD4D98BABEC13DDA3A2F5E8C3F1E2E3E2E3
```

> Note: Generate the thumbprint using the AWS guide if needed.

---

## Step 2: Create IAM Role with Conditions

Create a role that GitHub Actions can assume, scoped to specific resources.

### Create Trust Policy (JSON)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::ACCOUNT_ID:oidc-provider/token.actions.githubusercontent.com"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringLike": {
          "token.actions.githubusercontent.com:sub": "repo:YOUR_GITHUB_ORG/REPO_NAME"
        },
        "StringEquals": {
          "token.actions.githubusercontent.com:aud": "sts.amazonaws.com"
        }
      }
    }
  ]
}
```

Replace:
- `ACCOUNT_ID` — Your AWS account ID
- `YOUR_GITHUB_ORG` — Your GitHub organization or username
- `REPO_NAME` — Your repository name

### Create Inline Policy for ECR and ECS Access

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ecr:GetAuthorizationToken",
        "ecr:BatchCheckLayerAvailability",
        "ecr:GetDownloadUrlForLayer",
        "ecr:BatchGetImage",
        "ecr:PutImage",
        "ecr:InitiateLayerUpload",
        "ecr:UploadLayerPart",
        "ecr:CompleteLayerUpload"
      ],
      "Resource": "arn:aws:ecr:REGION:ACCOUNT_ID:repository/REPO_NAME"
    },
    {
      "Effect": "Allow",
      "Action": [
        "ecs:RegisterTaskDefinition",
        "ecs:DescribeTaskDefinition",
        "ecs:DescribeServices",
        "ecs:UpdateService",
        "ecs:DescribeClusters"
      ],
      "Resource": [
        "arn:aws:ecs:REGION:ACCOUNT_ID:cluster/CLUSTER_NAME",
        "arn:aws:ecs:REGION:ACCOUNT_ID:service/CLUSTER_NAME/SERVICE_NAME"
      ]
    },
    {
      "Effect": "Allow",
      "Action": "logs:CreateLogGroup",
      "Resource": "arn:aws:logs:REGION:ACCOUNT_ID:*"
    }
  ]
}
```

Replace:
- `REGION` — e.g., `us-west-2`
- `ACCOUNT_ID` — Your AWS account ID
- `REPO_NAME` — Your ECR repository name
- `CLUSTER_NAME` — Your ECS cluster name
- `SERVICE_NAME` — Your ECS service name

---

## Step 3: Configure GitHub Secrets

Add these secrets to your GitHub repository:

| Secret | Value | Example |
|--------|-------|---------|
| `AWS_ROLE_ARN` | Role ARN | `arn:aws:iam::123456789:role/github-actions-deploy-role` |
| `AWS_REGION` | AWS region | `us-west-2` |
| `ECR_REGISTRY` | ECR repository URL | `123456789012.dkr.ecr.us-west-2.amazonaws.com/python-container` |

### Adding Secrets

1. Go to your repository on GitHub
2. Navigate to **Settings → Secrets and variables → Actions**
3. Click "New repository secret"

---

## Step 4: Configure GitHub Variables

Add these variables (not secrets):

| Variable | Value | Example |
|----------|-------|---------|
| `ECS_CLUSTER` | ECS cluster name | `python-container-cluster` |
| `ECS_SERVICE` | ECS service name | `python-container` |

### Adding Variables

1. Go to your repository on GitHub
2. Navigate to **Settings → Secrets and variables → Actions**
3. Click "Variables" tab
4. Click "New variable"

---

## Restricting to Specific Repos

The trust policy condition restricts which GitHub repos can assume the role:

```json
"Condition": {
  "StringLike": {
    "token.actions.githubusercontent.com:sub": "repo:YOUR_ORG/repo-name"
  }
}
```

### Examples

| Pattern | Matches |
|---------|---------|
| `repo:my-org/python-container` | Only python-container |
| `repo:my-org/prefix-*` | Any repo starting with "prefix-" |

---

## Creating ECR Repository

If you need to create the ECR repository:

```bash
aws ecr create-repository \
  --repository-name python-container \
  --region us-west-2 \
  --image-scanning-configuration scanOnPush=true \
  --image-tag-mutability MUTABLE
```

---

## Creating ECS Cluster

If you need to create the ECS cluster:

```bash
aws ecs create-cluster \
  --cluster-name python-container-cluster \
  --capacity-providers FARGATE
```

---

## Troubleshooting

### Role Not Assumable

- Verify OIDC provider is created
- Check trust policy conditions match your repo

### Access Denied

- Verify inline policy has correct resource ARNs
- Check region matches where resources exist

### ECR Login Failed

- Ensure ECR repository exists
- Verify role has `ecr:GetAuthorizationToken` permission

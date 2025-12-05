# Cloudflare Pages Deploy

Builds and deploys applications to Cloudflare Pages with multi-environment support.

## Reference

```yaml
uses: Plattar/workflows/.github/workflows/cloudflare-pages-deploy.yml@main
```

## Overview

This workflow provides automated deployments to Cloudflare Pages with support for staging, review, and production environments. It can be triggered by tag pushes or manual workflow dispatch.

## Caller Workflow

Create `.github/workflows/deploy.yml` in your repository:

```yaml
name: Deploy
on:
  push:
    tags:
      - '*-staging'
      - '*-production'
      - '*-review'
      - '*'
  workflow_dispatch:
    inputs:
      tag:
        description: 'Tag to deploy (e.g., 1.0.0)'
        required: true
        type: string
      environment:
        description: 'Target deployment environment'
        required: true
        default: 'staging'
        type: choice
        options:
          - staging
          - production
          - review
concurrency:
  group: deploy-${{ github.event.inputs.environment || 'auto' }}
  cancel-in-progress: false
jobs:
  deploy:
    uses: Plattar/workflows/.github/workflows/cloudflare-pages-deploy.yml@main
    with:
      tag: ${{ github.event_name == 'workflow_dispatch' && github.event.inputs.tag || github.ref_name }}
      environment: ${{ github.event.inputs.environment || '' }}
      trigger-type: ${{ github.event_name == 'workflow_dispatch' && 'manual' || 'tag-push' }}
      project-name: 'your-project-name'
      build-directory: '.'
      output-directory: 'dist'
      build-command: 'npm run build'
    secrets:
      CLOUDFLARE_API_TOKEN: ${{ secrets.CLOUDFLARE_API_TOKEN }}
      CLOUDFLARE_ACCOUNT_ID: ${{ secrets.CLOUDFLARE_ACCOUNT_ID }}
```

## Required Secrets

Configure these in your repository under **Settings > Secrets and Variables > Actions**:

| Secret | Description |
|--------|-------------|
| `CLOUDFLARE_API_TOKEN` | Cloudflare API token with Pages deployment permissions |
| `CLOUDFLARE_ACCOUNT_ID` | Cloudflare account identifier |

### Obtaining Cloudflare Credentials

1. Log in to the [Cloudflare Dashboard](https://dash.cloudflare.com/)
2. Navigate to **My Profile > API Tokens**
3. Create a token with **Cloudflare Pages: Edit** permissions
4. Copy your Account ID from the dashboard URL or account overview

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `tag` | Yes | — | Tag to deploy |
| `trigger-type` | Yes | — | Trigger type: `tag-push` or `manual` |
| `project-name` | Yes | — | Cloudflare Pages project name |
| `environment` | No | — | Target environment: `staging`, `review`, or `production` |
| `domain-suffix` | No | `plattar.com` | Domain suffix for custom domains |
| `build-directory` | No | `.` | Directory containing package.json |
| `output-directory` | No | `build` | Build output path |
| `build-command` | No | `npm run clean:build` | Build command to execute |
| `node-version` | No | `latest` | Node.js version |
| `artifact-retention-days` | No | `120` | Days to retain build artifacts |

## Tag Conventions

The workflow determines the deployment target based on tag suffix:

| Tag Format | Action |
|------------|--------|
| `1.0.0` | Build only (no deployment) |
| `1.0.0-staging` | Build and deploy to staging |
| `1.0.0-review` | Build and deploy to review |
| `1.0.0-production` | Build and deploy to production |

## Examples

### Deploy to Staging

```bash
git tag 1.0.0-staging
git push origin 1.0.0-staging
```

### Deploy to Production

```bash
git tag 1.0.0-production
git push origin 1.0.0-production
```

### Manual Deployment via GitHub UI

1. Navigate to **Actions** tab in your repository
2. Select the **Deploy** workflow
3. Click **Run workflow**
4. Enter the tag version and select the target environment
5. Click **Run workflow**

## Troubleshooting

### Build Fails with "Project not found"

Ensure the `project-name` matches your Cloudflare Pages project exactly (case-sensitive).

### Deployment Succeeds but Site Not Updated

Check the `output-directory` matches your build tool's output path.

### Permission Denied Errors

Verify your `CLOUDFLARE_API_TOKEN` has the correct permissions for Cloudflare Pages deployments.
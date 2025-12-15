# Cloudflare Pages Deploy

Builds and deploys applications to Cloudflare Pages with multi-environment support. Supports both compiled applications and pre-built static files.

## Reference

```yaml
uses: Plattar/workflows/.github/workflows/cloudflare-pages-deploy.yml@main
```

## Overview

This workflow provides automated deployments to Cloudflare Pages with support for staging, review, and production environments. It can be triggered by tag pushes or manual workflow dispatch.

**Deploy Modes:**
- **Build Mode** (default): Compiles your application using npm/node before deployment
- **Static Mode**: Deploys pre-built static files directly without a build step

## Caller Workflow Examples

### Standard Build Deployment

Create `.github/workflows/deploy.yml` for projects that need compilation:

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
      deploy-mode: 'build'
      build-directory: '.'
      output-directory: 'dist'
      build-command: 'npm run build'
    secrets:
      CLOUDFLARE_API_TOKEN: ${{ secrets.CLOUDFLARE_API_TOKEN }}
      CLOUDFLARE_ACCOUNT_ID: ${{ secrets.CLOUDFLARE_ACCOUNT_ID }}
```

### Static Files Deployment

Create `.github/workflows/deploy.yml` for deploying pre-built static files:

```yaml
name: Deploy Static
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
      project-name: 'your-static-site'
      deploy-mode: 'static'
      static-source-directory: 'themes'
    secrets:
      CLOUDFLARE_API_TOKEN: ${{ secrets.CLOUDFLARE_API_TOKEN }}
      CLOUDFLARE_ACCOUNT_ID: ${{ secrets.CLOUDFLARE_ACCOUNT_ID }}
```

## Static Files Deployment

When using `deploy-mode: 'static'`, the workflow deploys pre-built files directly to Cloudflare Pages without running any build commands.

### Directory Structure Preservation

The entire contents of `static-source-directory` are deployed, preserving the full directory structure. For example:

**Repository structure:**
```
your-repo/
├── themes/
│   ├── webxr/
│   │   └── www/
│   │       ├── index.html
│   │       ├── styles.css
│   │       └── app.js
│   └── default/
│       └── www/
│           └── index.html
└── README.md
```

**With configuration:**
```yaml
static-source-directory: 'themes'
```

**Results in URLs:**
- `https://your-domain.com/webxr/www/index.html`
- `https://your-domain.com/webxr/www/styles.css`
- `https://your-domain.com/webxr/www/app.js`
- `https://your-domain.com/default/www/index.html`

### Root-Level Static Deployment

To deploy static files at the root (no subdirectory):

```yaml
static-source-directory: 'public'
```

**Repository structure:**
```
your-repo/
├── public/
│   ├── index.html
│   ├── styles.css
│   └── assets/
│       └── logo.png
└── README.md
```

**Results in URLs:**
- `https://your-domain.com/index.html`
- `https://your-domain.com/styles.css`
- `https://your-domain.com/assets/logo.png`

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

### Deployment Parameters

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `tag` | Yes | — | Tag to deploy |
| `trigger-type` | Yes | — | Trigger type: `tag-push` or `manual` |
| `project-name` | Yes | — | Cloudflare Pages project name |
| `environment` | No | — | Target environment: `staging`, `review`, or `production` |
| `domain-suffix` | No | `plattar.com` | Domain suffix for custom domains |

### Deploy Mode Configuration

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `deploy-mode` | No | `build` | Deployment mode: `build` or `static` |
| `static-source-directory` | No | `.` | Source directory for static files (only used when `deploy-mode` is `static`) |

### Build Configuration (only used when `deploy-mode` is `build`)

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `build-directory` | No | `.` | Directory containing package.json |
| `output-directory` | No | `build` | Build output path |
| `build-command` | No | `npm run clean:build` | Build command to execute |
| `node-version` | No | `latest` | Node.js version |

### Artifact Configuration

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `artifact-retention-days` | No | `120` | Days to retain build artifacts |

## Tag Conventions

The workflow determines the deployment target based on tag suffix:

| Tag Format | Action |
|------------|--------|
| `1.0.0` | Build/package only (no deployment) |
| `1.0.0-staging` | Build/package and deploy to staging |
| `1.0.0-review` | Build/package and deploy to review |
| `1.0.0-production` | Build/package and deploy to production |

## Examples

### Deploy Static Files to Staging

```bash
git tag 1.0.0-staging
git push origin 1.0.0-staging
```

### Deploy Static Files to Production

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

Check the `output-directory` (build mode) or `static-source-directory` (static mode) matches your actual file paths.

### Permission Denied Errors

Verify your `CLOUDFLARE_API_TOKEN` has the correct permissions for Cloudflare Pages deployments.

### Static Source Directory Not Found

Ensure the `static-source-directory` path exists in your repository and is relative to the repository root. The workflow will fail with a descriptive error if the directory doesn't exist or is empty.

### Files Not at Expected URLs

Remember that the entire contents of `static-source-directory` are deployed. If you set `static-source-directory: 'themes'` and have `themes/webxr/www/index.html`, the URL will be `{domain}/webxr/www/index.html`, not `{domain}/themes/webxr/www/index.html`.
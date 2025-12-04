# Shared Workflows

Reusable GitHub Actions workflows for Plattar repositories.

## Available Workflows

### Cloudflare Pages Deploy

Builds and deploys applications to Cloudflare Pages with multi-environment support.

**Reference:**
```yaml
uses: Plattar/workflows/.github/workflows/cloudflare-pages-deploy.yml@main
```

### ECR Build & Publish

Builds and publishes Docker images to AWS ECR. Designed for TypeScript projects.

**Reference:**
```yaml
uses: Plattar/workflows/.github/workflows/ecr-publish.yml@main
```

---

## Cloudflare Pages Deploy

### Caller Workflow

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

### Required Secrets

Configure these in your repository under **Settings > Secrets and Variables > Actions**:

| Secret | Description |
|--------|-------------|
| `CLOUDFLARE_API_TOKEN` | Cloudflare API token with Pages deployment permissions |
| `CLOUDFLARE_ACCOUNT_ID` | Cloudflare account identifier |

### Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `tag` | Yes | - | Tag to deploy |
| `trigger-type` | Yes | - | `tag-push` or `manual` |
| `project-name` | Yes | - | Cloudflare Pages project name |
| `environment` | No | - | Target environment (staging/review/production) |
| `domain-suffix` | No | `plattar.com` | Domain suffix for custom domains |
| `build-directory` | No | `.` | Directory containing package.json |
| `output-directory` | No | `build` | Build output path |
| `build-command` | No | `npm run clean:build` | Build command to execute |
| `node-version` | No | `latest` | Node.js version |
| `artifact-retention-days` | No | `120` | Days to retain build artifacts |

### Tag Conventions

| Format | Action |
|--------|--------|
| `1.0.0` | Build only |
| `1.0.0-staging` | Build and deploy to staging |
| `1.0.0-review` | Build and deploy to review |
| `1.0.0-production` | Build and deploy to production |

---

## ECR Build & Publish

Builds Docker images and publishes them to AWS ECR. Designed for TypeScript projects - automatically updates the version file at `{repository}/src/version.ts` with the release version from the git tag.

### Caller Workflow

Create `.github/workflows/publish.yml` in your repository:
```yaml
name: ECR Publish
on:
  push:
    tags:
      - '*.*.*'
jobs:
  publish:
    uses: Plattar/workflows/.github/workflows/ecr-publish.yml@main
    with:
      repository: your-service-name
    secrets:
      AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
      NPM_PUBLISH_KEY: ${{ secrets.NPM_PUBLISH_KEY }}
```

### Required Secrets

Configure these in your repository under **Settings > Secrets and Variables > Actions**:

| Secret | Description |
|--------|-------------|
| `AWS_ACCESS_KEY_ID` | AWS Access Key ID with permissions for ECR |
| `AWS_SECRET_ACCESS_KEY` | AWS Secret Access Key |

### Optional Secrets

| Secret | Description |
|--------|-------------|
| `NPM_PUBLISH_KEY` | NPM token for private packages |
| `DOCKER_BUILD_SECRETS` | Additional secrets for Docker build as KEY=VALUE pairs, newline separated |

### Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `repository` | Yes | - | Name of the repository/service. Used for ECR repository name, AWS role session name, and version.ts file path |
| `aws-region` | No | `ap-southeast-2` | AWS region for ECR |
| `dockerfile-path` | No | `.` | Path to Dockerfile context directory |
| `docker-build-args` | No | - | Additional Docker build arguments (multiline, one per line: ARG_NAME=value) |

### Outputs

| Output | Description |
|--------|-------------|
| `image-uri` | Full URI of the published Docker image in ECR |
| `image-tag` | Tag/version of the published image |

### Tag Conventions

Tags must follow semantic versioning format:

| Format | Example |
|--------|---------|
| `MAJOR.MINOR.PATCH` | `1.0.0`, `2.1.3` |

### Using Outputs

You can use the workflow outputs in subsequent jobs:
```yaml
name: ECR Publish
on:
  push:
    tags:
      - '*.*.*'
jobs:
  publish:
    uses: Plattar/workflows/.github/workflows/ecr-publish.yml@main
    with:
      repository: your-service-name
    secrets:
      AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}

  post-publish:
    needs: publish
    runs-on: ubuntu-latest
    steps:
      - name: Use published image info
        run: |
          echo "Published image: ${{ needs.publish.outputs.image-uri }}"
          echo "Version: ${{ needs.publish.outputs.image-tag }}"
```
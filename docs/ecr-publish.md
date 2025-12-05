# ECR Build & Publish

Builds Docker images and publishes them to AWS ECR. Designed for TypeScript projects.

## Reference

```yaml
uses: Plattar/workflows/.github/workflows/ecr-publish.yml@main
```

## Overview

This workflow automates the process of building Docker images and publishing them to Amazon Elastic Container Registry (ECR). It's specifically designed for TypeScript projects and automatically updates the version file at `{repository}/src/version.ts` with the release version from the git tag.

## Caller Workflow

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

## Required Secrets

Configure these in your repository under **Settings > Secrets and Variables > Actions**:

| Secret | Description |
|--------|-------------|
| `AWS_ACCESS_KEY_ID` | AWS Access Key ID with permissions for ECR |
| `AWS_SECRET_ACCESS_KEY` | AWS Secret Access Key |

## Optional Secrets

| Secret | Description |
|--------|-------------|
| `NPM_PUBLISH_KEY` | NPM token for private packages (if your Dockerfile needs private npm packages) |
| `DOCKER_BUILD_SECRETS` | Additional secrets for Docker build as KEY=VALUE pairs, newline separated |

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `repository` | Yes | — | Name of the repository/service. Used for ECR repository name, AWS role session name, and version.ts file path |
| `aws-region` | No | `ap-southeast-2` | AWS region for ECR |
| `dockerfile-path` | No | `.` | Path to Dockerfile context directory |
| `docker-build-args` | No | — | Additional Docker build arguments (multiline, one per line: `ARG_NAME=value`) |

## Outputs

| Output | Description |
|--------|-------------|
| `image-uri` | Full URI of the published Docker image in ECR |
| `image-tag` | Tag/version of the published image |

## Tag Conventions

Tags must follow semantic versioning format:

| Format | Example |
|--------|---------|
| `MAJOR.MINOR.PATCH` | `1.0.0`, `2.1.3`, `0.9.15` |

## Using Outputs in Subsequent Jobs

You can use the workflow outputs in subsequent jobs for deployments or notifications:

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
  deploy:
    needs: publish
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to ECS
        run: |
          echo "Deploying image: ${{ needs.publish.outputs.image-uri }}"
          echo "Version: ${{ needs.publish.outputs.image-tag }}"
          # Add your deployment commands here
  notify:
    needs: publish
    runs-on: ubuntu-latest
    steps:
      - name: Send Slack notification
        run: |
          echo "Published ${{ needs.publish.outputs.image-tag }} to ECR"
```

## Version File

The workflow automatically updates `{repository}/src/version.ts` with the release version. Ensure your project has this file structure:

```
your-service/
├── src/
│   └── version.ts
├── Dockerfile
└── package.json
```

Example `version.ts`:

```typescript
export default '0.0.0'; // This will be updated by the workflow
```

## Docker Build Arguments

Pass additional build arguments to Docker:

```yaml
with:
  repository: your-service-name
  docker-build-args: |
    NODE_ENV=production
    API_URL=https://api.example.com
```

## Docker Build Secrets

For sensitive values needed during build (not baked into the image):

```yaml
secrets:
  AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
  AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
  DOCKER_BUILD_SECRETS: |
    PRIVATE_KEY=${{ secrets.PRIVATE_KEY }}
    DATABASE_URL=${{ secrets.DATABASE_URL }}
```

Access these in your Dockerfile using:

```dockerfile
RUN --mount=type=secret,id=PRIVATE_KEY \
    cat /run/secrets/PRIVATE_KEY > /app/private.key
```

## Examples

### Basic Usage

```yaml
jobs:
  publish:
    uses: Plattar/workflows/.github/workflows/ecr-publish.yml@main
    with:
      repository: my-api
    secrets:
      AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

### With Private NPM Packages

```yaml
jobs:
  publish:
    uses: Plattar/workflows/.github/workflows/ecr-publish.yml@main
    with:
      repository: my-api
    secrets:
      AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
      NPM_PUBLISH_KEY: ${{ secrets.NPM_PUBLISH_KEY }}
```

### Custom AWS Region

```yaml
jobs:
  publish:
    uses: Plattar/workflows/.github/workflows/ecr-publish.yml@main
    with:
      repository: my-api
      aws-region: us-east-1
    secrets:
      AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

### Nested Dockerfile

```yaml
jobs:
  publish:
    uses: Plattar/workflows/.github/workflows/ecr-publish.yml@main
    with:
      repository: my-api
      dockerfile-path: ./docker/production
    secrets:
      AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

## Triggering a Release

```bash
# Create and push a semantic version tag
git tag 1.0.0
git push origin 1.0.0
```

## Troubleshooting

### "Repository does not exist" Error

Ensure the ECR repository exists in your AWS account. The workflow does not create repositories automatically.

### Authentication Failed

Verify your `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` have the following permissions:
- `ecr:GetAuthorizationToken`
- `ecr:BatchCheckLayerAvailability`
- `ecr:GetDownloadUrlForLayer`
- `ecr:BatchGetImage`
- `ecr:PutImage`
- `ecr:InitiateLayerUpload`
- `ecr:UploadLayerPart`
- `ecr:CompleteLayerUpload`

### Version File Not Updated

Ensure `{repository}/src/version.ts` exists and is writable.
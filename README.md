# Shared Workflows

Reusable GitHub Actions workflows for Plattar repositories.

## Available Workflows

| Workflow | Description | Documentation |
|----------|-------------|---------------|
| [Cloudflare Pages Deploy](.github/workflows/cloudflare-pages-deploy.yml) | Multi-environment deployments to Cloudflare Pages | [Guide](docs/cloudflare-pages-deploy.md) |
| [ECR Build & Publish](.github/workflows/ecr-publish.yml) | Docker images to AWS ECR for TypeScript projects | [Guide](docs/ecr-publish.md) |
| [NPM Publish](.github/workflows/npm-publish.yml) | NPM packages with S3/CDN support | [Guide](docs/npm-publish.md) |

## Quick Start

### Cloudflare Pages Deploy

```yaml
uses: Plattar/workflows/.github/workflows/cloudflare-pages-deploy.yml@main
```

### ECR Build & Publish

```yaml
uses: Plattar/workflows/.github/workflows/ecr-publish.yml@main
```

### NPM Publish

```yaml
uses: Plattar/workflows/.github/workflows/npm-publish.yml@main
```

## Required Secrets Overview

Configure secrets in your repository under **Settings > Secrets and Variables > Actions**.

| Workflow | Required Secrets | Optional Secrets |
|----------|------------------|------------------|
| Cloudflare Pages | `CLOUDFLARE_API_TOKEN`, `CLOUDFLARE_ACCOUNT_ID` | — |
| ECR Publish | `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY` | `NPM_PUBLISH_KEY`, `DOCKER_BUILD_SECRETS` |
| NPM Publish | `NPM_PUBLISH_KEY` | `AWS_S3_*` (for S3 upload) |

## Tag Conventions Summary

| Workflow | Tag Format | Example |
|----------|------------|---------|
| Cloudflare Pages | `VERSION-ENVIRONMENT` | `1.0.0-staging`, `1.0.0-production` |
| ECR Publish | `MAJOR.MINOR.PATCH` | `1.0.0`, `2.1.3` |
| NPM Publish | Semver with optional prerelease | `1.0.0`, `1.0.0-beta.1`, `1.0.0-alpha.1` |

See individual workflow documentation for complete details on inputs, outputs, and usage examples.

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Submit a pull request

## License

See [LICENSE](LICENSE) for details.
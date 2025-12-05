# NPM Publish

Builds and publishes packages to NPM with support for stable, beta, and alpha releases. Optionally syncs to AWS S3 and purges JSDelivr CDN cache.

## Reference

```yaml
uses: Plattar/workflows/.github/workflows/npm-publish.yml@main
```

## Overview

This workflow automates NPM package publishing for TypeScript projects. It automatically updates the version file at `{package-directory}/src/version.ts` with the release version from the git tag and supports multiple release channels (latest, beta, alpha) with appropriate NPM distribution tags.

## Caller Workflow

Create `.github/workflows/npm-publish.yml` in your repository:

```yaml
name: NPM Publish
on:
  push:
    tags:
      - '*.*.*'
      - '*.*.*-beta*'
      - '*.*.*-alpha*'
jobs:
  publish:
    uses: Plattar/workflows/.github/workflows/npm-publish.yml@main
    with:
      package-directory: 'your-package-directory'
      npm-scope: 'plattar'
      npm-access: 'public'
    secrets:
      NPM_PUBLISH_KEY: ${{ secrets.NPM_PUBLISH_KEY }}
```

## With S3 Upload and CDN Purge

For packages that need to be served via CDN:

```yaml
name: NPM Publish
on:
  push:
    tags:
      - '*.*.*'
      - '*.*.*-beta*'
      - '*.*.*-alpha*'
jobs:
  publish:
    uses: Plattar/workflows/.github/workflows/npm-publish.yml@main
    with:
      package-directory: 'plattar-ar-adapter'
      npm-scope: 'plattar'
      npm-access: 'public'
      enable-s3-upload: true
      s3-source-subdir: 'build/es2019'
      s3-dest-prefix: 'public-sdk'
      cdn-purge-urls: |
        https://cdn.jsdelivr.net/npm/@plattar/plattar-ar-adapter/build/es2015/plattar-ar-adapter.min.js
        https://cdn.jsdelivr.net/npm/@plattar/plattar-ar-adapter/build/es2019/plattar-ar-adapter.min.js
    secrets:
      NPM_PUBLISH_KEY: ${{ secrets.NPM_PUBLISH_KEY }}
      AWS_S3_BUCKET: ${{ secrets.AWS_S3_BUCKET }}
      AWS_S3_ACCESS_KEY_ID: ${{ secrets.AWS_S3_ACCESS_KEY_ID }}
      AWS_S3_SECRET_ACCESS_KEY: ${{ secrets.AWS_S3_SECRET_ACCESS_KEY }}
      AWS_S3_REGION: ${{ secrets.AWS_S3_REGION }}
```

## Required Secrets

Configure these in your repository under **Settings > Secrets and Variables > Actions**:

| Secret | Description |
|--------|-------------|
| `NPM_PUBLISH_KEY` | NPM authentication token for publishing |

### Obtaining NPM Token

1. Log in to [npmjs.com](https://www.npmjs.com/)
2. Navigate to **Access Tokens**
3. Generate a new **Automation** token
4. Copy the token and add it to your repository secrets

## Optional Secrets (S3 Upload)

Required only when `enable-s3-upload` is `true`:

| Secret | Description |
|--------|-------------|
| `AWS_S3_BUCKET` | AWS S3 bucket name |
| `AWS_S3_ACCESS_KEY_ID` | AWS access key ID |
| `AWS_S3_SECRET_ACCESS_KEY` | AWS secret access key |
| `AWS_S3_REGION` | AWS S3 region (e.g., `ap-southeast-2`) |

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `package-directory` | Yes | — | Directory containing the package to publish |
| `npm-scope` | Yes | — | NPM scope without @ symbol (e.g., `plattar`) |
| `node-version` | No | `latest` | Node.js version |
| `build-command` | No | `clean:build` | NPM script command to run for building |
| `npm-access` | No | `restricted` | NPM package access level: `public` or `restricted` |
| `copy-readme` | No | `true` | Copy root README.md to package directory |
| `copy-graphics` | No | `true` | Copy root graphics/ directory to package directory |
| `enable-s3-upload` | No | `false` | Enable AWS S3 upload |
| `s3-source-subdir` | No | `build/es2019` | Source subdirectory within package for S3 sync |
| `s3-dest-prefix` | No | `public-sdk` | S3 destination prefix path |
| `cdn-purge-urls` | No | — | Newline-separated list of JSDelivr URLs to purge |

## Outputs

| Output | Description |
|--------|-------------|
| `version` | The version that was published |
| `npm-tag` | The NPM tag used (`latest`, `beta`, or `alpha`) |
| `is-prerelease` | Whether this is a pre-release version (`true` or `false`) |

## Tag Conventions

Tags determine the NPM distribution tag used for publishing:

| Tag Format | NPM Tag | Example | Install Command |
|------------|---------|---------|-----------------|
| `MAJOR.MINOR.PATCH` | `latest` | `1.0.0` | `npm i @plattar/pkg` |
| `MAJOR.MINOR.PATCH-beta.N` | `beta` | `1.0.0-beta.1` | `npm i @plattar/pkg@beta` |
| `MAJOR.MINOR.PATCH-alpha.N` | `alpha` | `1.0.0-alpha.1` | `npm i @plattar/pkg@alpha` |

Beta and alpha releases are **not** included when users install with semver ranges (e.g., `^1.0.0`), ensuring pre-release versions do not automatically update production dependencies.

## Using Outputs in Subsequent Jobs

```yaml
name: NPM Publish
on:
  push:
    tags:
      - '*.*.*'
      - '*.*.*-beta*'
      - '*.*.*-alpha*'
jobs:
  publish:
    uses: Plattar/workflows/.github/workflows/npm-publish.yml@main
    with:
      package-directory: 'my-package'
      npm-scope: 'plattar'
      npm-access: 'public'
    secrets:
      NPM_PUBLISH_KEY: ${{ secrets.NPM_PUBLISH_KEY }}
  notify:
    needs: publish
    runs-on: ubuntu-latest
    steps:
      - name: Announce release
        run: |
          echo "Published version: ${{ needs.publish.outputs.version }}"
          echo "NPM tag: ${{ needs.publish.outputs.npm-tag }}"
          echo "Is pre-release: ${{ needs.publish.outputs.is-prerelease }}"
  deploy-docs:
    needs: publish
    if: needs.publish.outputs.is-prerelease == 'false'
    runs-on: ubuntu-latest
    steps:
      - name: Deploy documentation
        run: echo "Deploying docs for stable release ${{ needs.publish.outputs.version }}"
```

## Version File

The workflow automatically updates `{package-directory}/src/version.ts`. Ensure your package has this file:

```typescript
export default '0.0.0'; // Updated automatically by workflow
```

## Project Structure

Expected directory structure:

```
your-repo/
├── README.md                    # Copied to package if copy-readme: true
├── graphics/                    # Copied to package if copy-graphics: true
│   └── logo.png
└── your-package-directory/
    ├── package.json
    ├── src/
    │   ├── index.ts
    │   └── version.ts
    └── build/                   # Generated by build command
        ├── es2015/
        └── es2019/
```

## Examples

### Basic Public Package

```yaml
jobs:
  publish:
    uses: Plattar/workflows/.github/workflows/npm-publish.yml@main
    with:
      package-directory: 'sdk'
      npm-scope: 'plattar'
      npm-access: 'public'
    secrets:
      NPM_PUBLISH_KEY: ${{ secrets.NPM_PUBLISH_KEY }}
```

### Private Scoped Package

```yaml
jobs:
  publish:
    uses: Plattar/workflows/.github/workflows/npm-publish.yml@main
    with:
      package-directory: 'internal-tools'
      npm-scope: 'plattar'
      npm-access: 'restricted'
    secrets:
      NPM_PUBLISH_KEY: ${{ secrets.NPM_PUBLISH_KEY }}
```

### Custom Build Command

```yaml
jobs:
  publish:
    uses: Plattar/workflows/.github/workflows/npm-publish.yml@main
    with:
      package-directory: 'my-package'
      npm-scope: 'plattar'
      build-command: 'build:prod'
    secrets:
      NPM_PUBLISH_KEY: ${{ secrets.NPM_PUBLISH_KEY }}
```

### Without README/Graphics Copy

```yaml
jobs:
  publish:
    uses: Plattar/workflows/.github/workflows/npm-publish.yml@main
    with:
      package-directory: 'my-package'
      npm-scope: 'plattar'
      copy-readme: false
      copy-graphics: false
    secrets:
      NPM_PUBLISH_KEY: ${{ secrets.NPM_PUBLISH_KEY }}
```

## Triggering Releases

### Stable Release

```bash
git tag 1.0.0
git push origin 1.0.0
```

### Beta Release

```bash
git tag 1.0.0-beta.1
git push origin 1.0.0-beta.1
```

### Alpha Release

```bash
git tag 1.0.0-alpha.1
git push origin 1.0.0-alpha.1
```

## CDN Purge

When `cdn-purge-urls` is provided, the workflow automatically purges the JSDelivr CDN cache for those URLs after publishing. This ensures users receive the latest version immediately.

Example URLs to purge:

```yaml
cdn-purge-urls: |
  https://cdn.jsdelivr.net/npm/@plattar/sdk/build/es2019/sdk.min.js
  https://cdn.jsdelivr.net/npm/@plattar/sdk/build/es2015/sdk.min.js
```

## Troubleshooting

### "403 Forbidden" on Publish

- Verify `NPM_PUBLISH_KEY` has publish permissions
- For scoped packages, ensure you have access to the scope
- Check `npm-access` is set correctly (`public` for public packages)

### Version Already Exists

NPM does not allow publishing the same version twice. Increment your version and create a new tag.

### S3 Upload Fails

- Verify all `AWS_S3_*` secrets are configured
- Check IAM permissions for `s3:PutObject` and `s3:ListBucket`
- Ensure the bucket exists in the specified region

### CDN Not Updating

- Verify the URLs in `cdn-purge-urls` are correct
- JSDelivr purge can take a few minutes to propagate
- Check that the package version on NPM is updated first
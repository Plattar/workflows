# Generate Folder Markdown

Generates a comprehensive markdown file from folder contents, optimized for AI/LLM consumption. Useful for code analysis, improvements, documentation generation, or feeding entire codebases into AI assistants.

## Reference
````yaml
uses: Plattar/workflows/.github/workflows/generate-folder-markdown.yml@main
````

## Overview

This workflow scans a specified directory, extracts all text-based source files, and generates a single markdown document containing the complete codebase with syntax highlighting, file metadata, and a table of contents. The output is designed for uploading to AI/LLM tools for code review, refactoring suggestions, or feature development.

## Caller Workflow

Create `.github/workflows/generate-docs.yml` in your repository:
````yaml
name: Generate AI-Ready Docs
on:
  push:
    branches: [main]
  workflow_dispatch:
jobs:
  generate:
    uses: Plattar/workflows/.github/workflows/generate-folder-markdown.yml@main
    with:
      target_folder: 'src'
````

## With Custom Configuration

For projects requiring specific exclusions or output settings:
````yaml
name: Generate AI-Ready Docs
on:
  push:
    branches: [main, develop]
  workflow_dispatch:
    inputs:
      folder:
        description: 'Folder to scan'
        required: true
        default: 'src'
jobs:
  generate:
    uses: Plattar/workflows/.github/workflows/generate-folder-markdown.yml@main
    with:
      target_folder: ${{ github.event.inputs.folder || 'src' }}
      output_filename: 'codebase-for-ai'
      artifact_name: 'ai-ready-codebase'
      retention_days: 14
      excluded_dirs: 'tests,fixtures,__mocks__'
      excluded_extensions: '.test.js,.spec.ts,.stories.tsx'
      include_file_stats: true
      max_file_size_kb: 250
````

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `target_folder` | Yes | — | Directory to scan and generate markdown from |
| `output_filename` | No | `{folder}_contents` | Custom output filename without `.md` extension |
| `artifact_name` | No | `folder-contents-markdown` | Name for the uploaded artifact |
| `retention_days` | No | `30` | Number of days to retain the artifact |
| `excluded_dirs` | No | — | Comma-separated list of additional directories to exclude |
| `excluded_extensions` | No | — | Comma-separated list of additional file extensions to exclude |
| `include_file_stats` | No | `true` | Include line count and file statistics |
| `max_file_size_kb` | No | `500` | Maximum file size in KB to include |

## Outputs

| Output | Description |
|--------|-------------|
| `markdown_file` | The generated markdown filename |
| `file_count` | Number of files processed |
| `skipped_count` | Number of files skipped |
| `artifact_name` | Name of the uploaded artifact |

## Default Exclusions

The workflow automatically excludes common non-source directories and binary files.

### Excluded Directories

`node_modules`, `__pycache__`, `venv`, `env`, `.git`, `dist`, `build`, `coverage`, `.next`, `.nuxt`, `vendor`

### Excluded File Types

Binary files including images, videos, audio, fonts, archives, compiled files, and common non-text formats are automatically skipped.

## Supported Languages

The workflow provides syntax highlighting for 70+ file types including:

| Category | Extensions |
|----------|------------|
| JavaScript/TypeScript | `.js`, `.ts`, `.jsx`, `.tsx`, `.vue`, `.svelte` |
| Python | `.py` |
| Systems | `.c`, `.cpp`, `.h`, `.hpp`, `.rs`, `.go`, `.zig` |
| JVM | `.java`, `.kt`, `.scala`, `.clj` |
| Web | `.html`, `.css`, `.scss`, `.sass`, `.less` |
| Data | `.json`, `.yaml`, `.yml`, `.xml`, `.toml` |
| DevOps | `.dockerfile`, `.tf`, `.hcl`, `.sh`, `.bash` |
| Other | `.rb`, `.php`, `.swift`, `.dart`, `.sql`, `.graphql` |

## Using Outputs in Subsequent Jobs
````yaml
name: Generate and Process Docs
on:
  push:
    branches: [main]
jobs:
  generate:
    uses: Plattar/workflows/.github/workflows/generate-folder-markdown.yml@main
    with:
      target_folder: 'src'
      artifact_name: 'source-docs'
  process:
    needs: generate
    runs-on: ubuntu-latest
    steps:
      - name: Report statistics
        run: |
          echo "Processed files: ${{ needs.generate.outputs.file_count }}"
          echo "Skipped files: ${{ needs.generate.outputs.skipped_count }}"
          echo "Output file: ${{ needs.generate.outputs.markdown_file }}"
````

## Multiple Folders

Generate separate documents for different parts of your codebase:
````yaml
name: Generate Docs for All Modules
on:
  workflow_dispatch:
jobs:
  frontend:
    uses: Plattar/workflows/.github/workflows/generate-folder-markdown.yml@main
    with:
      target_folder: 'frontend/src'
      artifact_name: 'frontend-codebase'
  backend:
    uses: Plattar/workflows/.github/workflows/generate-folder-markdown.yml@main
    with:
      target_folder: 'backend/src'
      artifact_name: 'backend-codebase'
  shared:
    uses: Plattar/workflows/.github/workflows/generate-folder-markdown.yml@main
    with:
      target_folder: 'shared/lib'
      artifact_name: 'shared-lib-codebase'
````

## Generated Output Structure

The generated markdown file includes:

1. **Header** - Repository metadata, branch, commit SHA, and generation timestamp
2. **Table of Contents** - Linked list of all processed files
3. **File Sections** - Each file with metadata table and syntax-highlighted content
4. **Summary** - Statistics including file counts, line counts, and skip reasons

Example output structure:
````markdown
# Files from `src` Directory

> **Purpose:** This document contains the source code from the `src` directory, formatted for AI/LLM consumption.

**Generated:** 2024-01-15 10:30:00 UTC
**Repository:** `Plattar/my-project`
**Branch:** `main`
**Commit:** `abc1234`

---

## Table of Contents

- [`src/index.ts`](#file-srcindexts)
- [`src/utils/helpers.ts`](#file-srcutilshelpersts)

---

## File: `src/index.ts`

| Property | Value |
|----------|-------|
| Directory | `src` |
| Filename | `index.ts` |
| Extension | `.ts` |
| Lines | 150 |
| Characters | 4,230 |
```typescript
// File contents here
```

---

## Summary

### Statistics

| Metric | Value |
|--------|-------|
| Files processed | 42 |
| Files skipped | 8 |
| Total lines | 5,432 |
| Total characters | 156,789 |
````

## Examples

### Basic Source Directory
````yaml
jobs:
  generate:
    uses: Plattar/workflows/.github/workflows/generate-folder-markdown.yml@main
    with:
      target_folder: 'src'
````

### Monorepo Package
````yaml
jobs:
  generate:
    uses: Plattar/workflows/.github/workflows/generate-folder-markdown.yml@main
    with:
      target_folder: 'packages/core/src'
      output_filename: 'core-package'
      artifact_name: 'core-source'
````

### Exclude Test Files
````yaml
jobs:
  generate:
    uses: Plattar/workflows/.github/workflows/generate-folder-markdown.yml@main
    with:
      target_folder: 'src'
      excluded_dirs: '__tests__,__mocks__,fixtures'
      excluded_extensions: '.test.ts,.spec.ts,.test.tsx,.spec.tsx'
````

### Minimal Output
````yaml
jobs:
  generate:
    uses: Plattar/workflows/.github/workflows/generate-folder-markdown.yml@main
    with:
      target_folder: 'lib'
      include_file_stats: false
      max_file_size_kb: 100
````

### Short Retention
````yaml
jobs:
  generate:
    uses: Plattar/workflows/.github/workflows/generate-folder-markdown.yml@main
    with:
      target_folder: 'src'
      retention_days: 7
````

## Use Cases

### Code Review with AI

1. Trigger the workflow manually or on push
2. Download the generated markdown artifact
3. Upload to Claude, ChatGPT, or similar AI assistant
4. Request code review, security analysis, or improvement suggestions

### Documentation Generation

1. Generate markdown from your source code
2. Use AI to create API documentation, README files, or usage guides
3. Review and refine the generated documentation

### Refactoring Assistance

1. Generate a complete snapshot of your codebase
2. Describe the desired refactoring to an AI assistant
3. Receive implementation suggestions with full context

### Onboarding New Developers

1. Generate documentation of the entire codebase
2. Share with new team members for rapid familiarization
3. Use AI to answer questions about code structure and patterns

## Troubleshooting

### Folder Not Found

Ensure the `target_folder` path is relative to the repository root and exists in your repository.

### Files Not Appearing

Check if files are being excluded due to:
- File extension in the binary exclusion list
- Directory in the excluded directories list
- File size exceeding `max_file_size_kb`
- File not readable as UTF-8 text

Review the **Summary** section in the generated markdown for skip reasons.

### Output Too Large

If the generated file is too large for your AI tool:
- Reduce `max_file_size_kb` to exclude large files
- Add more directories to `excluded_dirs`
- Exclude test and story files via `excluded_extensions`
- Generate separate documents for different modules

### Missing Syntax Highlighting

If a file type lacks syntax highlighting, the language may not be in the supported list. The file will still be included with generic code block formatting.
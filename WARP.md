# WARP.md

This file provides guidance to WARP (warp.dev) when working with code in this repository.

## Overview

k6 documentation repository built with Hugo. Documentation is written in Markdown and published to https://grafana.com/docs/k6/. The repository maintains multiple versions of the documentation simultaneously.

## Local Development

Start the local development server:

```sh
npm start
```

Access the preview at http://localhost:3002/docs/k6/

**Requirements:**
- Docker or Podman (Docker Desktop must be running if using Docker)
- Node.js >=16.0.0
- npm >=7.0.0

## Multi-Version Documentation System

The repository maintains documentation for multiple k6 versions simultaneously:

- `docs/sources/k6/next/` - Upcoming major release (work-in-progress)
- `docs/sources/k6/v1.3.x/` - Latest stable version
- `docs/sources/k6/v1.2.x/`, `v1.1.x/`, etc. - Previous versions

**Critical workflow:** Changes to the latest stable version MUST also be applied to `next/`:

```sh
# Make changes in docs/sources/k6/v1.3.x/
# Then port to next using apply-patch script
scripts/apply-patch HEAD~ docs/sources/k6/v1.3.x docs/sources/k6/next

# For multiple commits:
scripts/apply-patch HEAD~3 docs/sources/k6/v1.3.x docs/sources/k6/next
```

Changes only for upcoming releases go directly in `docs/sources/k6/next/` without porting.

## Code Validation System

JavaScript code snippets in Markdown are automatically validated and run with k6:

**ESLint validation:**
- All JS code blocks are linted on commit via Husky pre-commit hooks
- Skip incomplete examples: `<!-- eslint-skip -->`

**k6 execution validation:**
- Code blocks run with `k6 run -w` on PRs (only for changed files in `docs/sources/next/`)
- Test locally: `python3 scripts/md-k6.py docs/sources/k6/next/path/to/file.md`

**Special directives (placed above code blocks):**
- `<!-- md-k6:skip -->` - Skip running this code block
- `<!-- md-k6:skipall -->` - Skip entire Markdown file (can be placed anywhere)
- `<!-- md-k6:nofail -->` - Allow errors in logs but fail on non-zero exit
- `<!-- md-k6:env.KEY=VALUE -->` - Set environment variables
- `<!-- md-k6:arg.--flag -->` - Pass CLI arguments to k6
- `<!-- md-k6:nothresholds -->` - Disable thresholds processing
- `<!-- md-k6:fixedscenarios -->` - Prevent scenario modification by CLI args

Multiple directives can be combined: `<!-- md-k6:skip,env.FOO=bar -->`

## Grafana Documentation Standards

This repository follows Grafana's strict documentation standards (see AGENTS.md for full details):

**Style requirements:**
- Sentence case for all headings
- Present simple tense, active voice, second person
- Write complete product names: "OpenTelemetry" (not "OTel"), "Kubernetes" (not "K8s")
- Use placeholders in UPPERCASE_WITH_UNDERSCORES: `<SERVICE_NAME>`
- Front matter `title` field must match H1 heading exactly
- Always include copy immediately after headings (no consecutive headings)

**Hugo shortcodes for admonitions:**
```markdown
{{< admonition type="note" >}}
Important information here
{{< /admonition >}}
```

Types: `note`, `caution`, `warning` (use sparingly)

Full style guide: https://grafana.com/docs/writers-toolkit/

## Quality Tools

**Pre-commit hooks (Husky + lint-staged):**
- Prettier formats Markdown
- ESLint validates JavaScript in code blocks

**Vale prose linting:**
- Configured for Grafana style guide
- Run via Writers' Toolkit (see Grafana documentation)

## Deployment

Changes in `docs/sources/` merged to `main` automatically deploy to production via GitHub Actions (workflow: `publish-technical-documentation.yml`).

Only changes within `docs/sources/` trigger deployment.

## Version Release Process

When k6 releases a new major version:

1. Review and merge open PRs for the next release
2. `git pull` latest
3. Duplicate `docs/sources/k6/next/` folder
4. Rename copy to match version (e.g., `v0.58.x`)
5. Build locally to verify new version appears
6. Commit and push

The latest numbered version automatically becomes the "latest" version displayed.

## Reusable Content

Shared Markdown files in `docs/sources/VERSION/shared/` can be included using:

```markdown
{{< docs/shared source="k6" lookup="file-name.md" version="<K6_VERSION>" >}}
```

## Repository-Specific Scripts

- `scripts/apply-patch` - Port git changes between version directories
- `scripts/md-k6.py` - Run k6 against code blocks in Markdown files
- `scripts/check-canonicals` - Validate canonical URLs

# Vercel Deploy Action

A reusable GitHub Action for deploying preview and production environments to Vercel with automatic PR comments and full build control.

## Usage

<details open>
<summary><strong>Basic Usage</strong></summary>

```yaml
name: Deploy Preview

on:
  pull_request:
    types: [opened, synchronize, reopened]

permissions:
  contents: read
  pull-requests: write

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - uses: W3Dev/vercel-deploy@main
        with:
          vercel_token: ${{ secrets.VERCEL_TOKEN }}
          vercel_org_id: 'team_xxxxx'
          vercel_project_id: 'prj_xxxxx'
```

</details>

<details>
<summary><strong>Preview Environment</strong></summary>

Deploy with the preview environment explicitly set. This is the default behavior, shown here for clarity.

```yaml
- uses: W3Dev/vercel-deploy@main
  with:
    vercel_token: ${{ secrets.VERCEL_TOKEN }}
    vercel_org_id: 'team_xxxxx'
    vercel_project_id: 'prj_xxxxx'
    environment: preview
    alias_prefix: 'myapp'
```

This creates preview deployments with stable aliases like `pr-123--myapp.vercel.app`.

</details>

<details>
<summary><strong>Production Deployment (on merge to main)</strong></summary>

```yaml
name: Deploy Production

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: W3Dev/vercel-deploy@main
        with:
          vercel_token: ${{ secrets.VERCEL_TOKEN }}
          vercel_org_id: 'team_xxxxx'
          vercel_project_id: 'prj_xxxxx'
          environment: production
```

</details>

<details>
<summary><strong>Custom Alias Domain</strong></summary>

Use a custom domain for preview aliases instead of the default `.vercel.app`.

**In PR deployments**, `alias_domain` replaces the `.vercel.app` suffix:

```yaml
- uses: W3Dev/vercel-deploy@main
  with:
    vercel_token: ${{ secrets.VERCEL_TOKEN }}
    vercel_org_id: 'team_xxxxx'
    vercel_project_id: 'prj_xxxxx'
    alias_prefix: 'myapp'
    alias_domain: 'preview.example.com'
```

This creates aliases like `pr-123--myapp.preview.example.com` instead of `pr-123--myapp.vercel.app`.

In PR deployments you can also set `alias_domain` without `alias_prefix` -- the repo name will be used as the prefix automatically:

```yaml
- uses: W3Dev/vercel-deploy@main
  with:
    vercel_token: ${{ secrets.VERCEL_TOKEN }}
    vercel_org_id: 'team_xxxxx'
    vercel_project_id: 'prj_xxxxx'
    alias_domain: 'preview.example.com'
```

This creates aliases like `pr-123--my-repo.preview.example.com`.

**Outside PR context** (e.g., `push` to main), `alias_domain` is used directly as the full alias target:

```yaml
name: Deploy to custom domain

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: W3Dev/vercel-deploy@main
        with:
          vercel_token: ${{ secrets.VERCEL_TOKEN }}
          vercel_org_id: 'team_xxxxx'
          vercel_project_id: 'prj_xxxxx'
          alias_domain: 'dev.example.vercel.app'
          environment: preview
```

This aliases the deployment directly to `dev.example.vercel.app`.

> **Note:** The custom domain must be configured in your Vercel project settings before use.

</details>

<details>
<summary><strong>Monorepo Usage</strong></summary>

```yaml
- uses: W3Dev/vercel-deploy@main
  with:
    vercel_token: ${{ secrets.VERCEL_TOKEN }}
    vercel_org_id: 'team_xxxxx'
    vercel_project_id: 'prj_xxxxx'
    working_directory: 'apps/dashboard'
    alias_prefix: 'myapp-dashboard'
```

</details>

<details>
<summary><strong>With Prebuild Script</strong></summary>

```yaml
- uses: W3Dev/vercel-deploy@main
  with:
    vercel_token: ${{ secrets.VERCEL_TOKEN }}
    vercel_org_id: 'team_xxxxx'
    vercel_project_id: 'prj_xxxxx'
    prebuild_script: |
      git config --global user.email "ci@example.com"
      git config --global user.name "CI Bot"
```

</details>

<details>
<summary><strong>Production with Pre-deploy Script</strong></summary>

The `predeploy_script` runs **after** `vercel build` but **before** `vercel deploy --prebuilt`. Use it to run checks or steps that require the built output to exist, or to gate the actual upload to Vercel.

```yaml
- uses: W3Dev/vercel-deploy@main
  with:
    vercel_token: ${{ secrets.VERCEL_TOKEN }}
    vercel_org_id: 'team_xxxxx'
    vercel_project_id: 'prj_xxxxx'
    environment: production
    predeploy_script: |
      echo "Running post-build checks before deploying..."
      npm run test
```

> **Tip:** To run scripts **before** the build (e.g., code generation or configuring git), use `prebuild_script` instead.

</details>

<details>
<summary><strong>Using Yarn</strong></summary>

```yaml
- uses: W3Dev/vercel-deploy@main
  with:
    vercel_token: ${{ secrets.VERCEL_TOKEN }}
    vercel_org_id: 'team_xxxxx'
    vercel_project_id: 'prj_xxxxx'
    package_manager: 'yarn'
```

</details>

<details>
<summary><strong>Using pnpm</strong></summary>

```yaml
- uses: W3Dev/vercel-deploy@main
  with:
    vercel_token: ${{ secrets.VERCEL_TOKEN }}
    vercel_org_id: 'team_xxxxx'
    vercel_project_id: 'prj_xxxxx'
    package_manager: 'pnpm'
```

</details>

<details>
<summary><strong>Passing Additional Deploy Arguments</strong></summary>

Pass `deploy_args` as a single-line argument string such as `--archive=tgz --meta key=value`.
Quoted values are supported when needed, for example `--meta "description=My App"`.

```yaml
- uses: W3Dev/vercel-deploy@main
  with:
    vercel_token: ${{ secrets.VERCEL_TOKEN }}
    vercel_org_id: 'team_xxxxx'
    vercel_project_id: 'prj_xxxxx'
    deploy_args: '--archive=tgz'
```

</details>

<details>
<summary><strong>Teardown on PR Close</strong></summary>

Clean up preview deployments automatically when a PR is closed (merged or abandoned):

```yaml
name: Deploy Preview

on:
  pull_request:
    types: [opened, synchronize, reopened, closed]  # Include 'closed' for teardown

permissions:
  contents: read
  pull-requests: write

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: W3Dev/vercel-deploy@main
        with:
          vercel_token: ${{ secrets.VERCEL_TOKEN }}
          vercel_org_id: 'team_xxxxx'
          vercel_project_id: 'prj_xxxxx'
          alias_prefix: 'myapp'
          teardown: 'true'
          auto_clean_deployment: 'true'  # Also delete deployment, not just alias
```

**How it works:**
- When `teardown: 'true'` and the PR is closed, the action removes the preview alias
- With `auto_clean_deployment: 'true'`, it also deletes the deployment itself from Vercel
- The PR comment is updated to reflect the cleanup
- When the PR is open (opened/synchronize/reopened), normal deployment occurs

**Teardown modes:**
| Mode | `teardown` | `auto_clean_deployment` | Behavior |
|------|------------|------------------------|----------|
| No cleanup | `false` | - | Alias remains after PR close |
| Alias only | `true` | `false` | Removes alias, keeps deployment |
| Full cleanup | `true` | `true` | Removes alias AND deletes deployment |

</details>

<details>
<summary><strong>Full Workflow Example</strong></summary>

```yaml
name: Dashboard Preview

on:
  pull_request:
    types: [opened, synchronize, reopened]
    paths:
      - 'apps/dashboard/**'
  workflow_dispatch:
    inputs:
      pr_number:
        description: 'PR number to deploy'
        required: false

permissions:
  contents: read
  pull-requests: write

jobs:
  deploy:
    runs-on: self-hosted
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      
      - uses: W3Dev/vercel-deploy@main
        id: deploy
        with:
          vercel_token: ${{ secrets.VERCEL_TOKEN }}
          vercel_org_id: 'team_xxxxx'
          vercel_project_id: 'prj_xxxxx'
          package_manager: 'bun'
          working_directory: 'apps/dashboard'
          alias_prefix: 'myapp'
      
      - name: Use deployment URL
        run: echo "Deployed to ${{ steps.deploy.outputs.deployment_url }}"
```

</details>

## Why Use This Instead of Vercel's Native Integration?

While Vercel provides native GitHub integration, this action gives you **control over where and how your builds happen**:

### Key Advantages

**Build in Your GitHub Actions Environment**
- Builds run on **your GitHub Actions runners**, not Vercel's infrastructure
- Perfect for self-hosted runners with better caching, specific tooling, or compliance requirements
- Use `vercel build` + `vercel deploy --prebuilt` for full control

**Advanced Build Customization**
- Run prebuild scripts (configure git, generate code, set up credentials)
- Override install commands for complex dependency requirements
- Pass additional deploy flags such as `--archive=tgz`
- Native Bun, Yarn, npm, and pnpm support

**Stable Preview URLs**
- Get predictable PR-specific URLs: `pr-123--myapp.vercel.app`
- Use custom domains for aliases: `pr-123--myapp.preview.example.com`
- Vercel's native integration generates random deployment URLs
- Easier to share and reference in testing workflows

**Monorepo-Friendly**
- Explicit `working_directory` support for monorepos
- Fine-grained control over which parts of your repo to deploy

**Smart PR Comments**
- Updates the **same comment** on subsequent pushes (no PR spam)
- Vercel's native integration can create multiple comments per PR

**Workflow Flexibility**
- Integrate with larger workflows (run tests first, conditional deploys, etc.)
- Manual deployment triggers with `workflow_dispatch`

### When to Use Vercel's Native Integration Instead

Use Vercel's native integration if you:
- Want zero configuration
- Don't need custom build steps or environment control
- Prefer Vercel to manage the entire build environment

## Features

- Build in GitHub Actions, deploy to Vercel
- Support for both preview and production deployments
- Stable preview aliases (`pr-123--myapp.vercel.app`)
- Custom alias domains (`pr-123--myapp.preview.example.com`)
- Smart PR comments that update instead of spam
- Support for Bun, Yarn, npm, and pnpm
- Prebuild and predeploy scripts for custom workflows
- Monorepo support with `working_directory`
- GitHub Actions summary
- Automatic teardown of preview deployments on PR close

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `vercel_token` | Vercel API token | Yes | - |
| `vercel_org_id` | Vercel Organization/Team ID | Yes | - |
| `vercel_project_id` | Vercel Project ID | Yes | - |
| `vercel_project_name` | Project name for linking | No | repo name |
| `package_manager` | `bun`, `yarn`, `npm`, or `pnpm` | No | `bun` |
| `node_version` | Node.js version | No | `22` |
| `working_directory` | Build directory | No | `.` |
| `alias_prefix` | Prefix for preview alias (e.g., `myapp` creates `pr-123--myapp.vercel.app`) | No | - |
| `alias_domain` | Custom domain for preview alias. In PR context, replaces `.vercel.app` suffix. Outside PR context, used directly as the full alias target | No | - |
| `prebuild_script` | Script to run before `vercel build` | No | - |
| `predeploy_script` | Script to run after `vercel build` but before `vercel deploy --prebuilt` | No | - |
| `install_command` | Custom install command | No | - |
| `deploy_args` | Space-separated extra arguments for `vercel deploy --prebuilt` | No | - |
| `environment` | Deployment environment (`preview` or `production`) | No | `preview` |
| `github_token` | Token for PR comments | No | `github.token` |
| `teardown` | Remove preview alias when PR is closed | No | `false` |
| `auto_clean_deployment` | Also delete the deployment (not just alias) on teardown | No | `false` |

## Outputs

| Output | Description |
|--------|-------------|
| `deployment_url` | The Vercel deployment URL |
| `alias_url` | The aliased preview URL |
| `pr_number` | The PR number (if applicable) |

## License

MIT

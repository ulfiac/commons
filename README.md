# commons

This repository serves as a centralized hub for reusable GitHub Actions workflows within the ulfiac organization. It provides standardized, maintainable CI/CD components that ensure consistency across multiple repositories.

## Purpose

The primary goal is to reduce duplication and maintain high code quality standards by offering shared workflows that can be easily integrated into any repository. This approach simplifies maintenance, ensures security best practices, and allows for centralized updates to CI/CD processes.

## Tool Version Management

Tool versions are managed using `mise.toml` at the repository root to ensure consistent environments across different systems and CI runs.

## Reusable Workflows

### reusable_linter.yaml

A comprehensive linting workflow that performs multiple code quality checks. This workflow can be called from other repositories to validate code before merging.

#### Inputs

| Input | Type | Default | Description |
|-------|------|---------|-------------|
| `actionlint` | boolean | `true` | Enable GitHub Actions workflow linting |
| `shellcheck` | boolean | `true` | Enable shell script linting |
| `terraform-docs` | boolean | `false` | Enable Terraform documentation validation (checks for drift) |
| `terraform-fmt` | boolean | `true` | Enable Terraform formatting validation |
| `terragrunt-hcl-fmt` | boolean | `true` | Enable HCL formatting validation using Terragrunt |
| `tflint` | boolean | `true` | Enable Terraform static analysis |
| `trivy-config` | boolean | `true` | Enable security scanning of configuration files |

#### Usage Example

To use this workflow in another repository, add the following to your `.github/workflows/linter.yaml`:

```yaml
name: linter
run-name: "linter: ${{ github.event_name == 'pull_request' && github.event.pull_request.title || github.ref_name }}"

on:
  pull_request:
    branches:
      - main
  workflow_dispatch:

permissions:
  contents: read

jobs:
  reusable-linter:
    uses: ulfiac/commons/.github/workflows/reusable_linter.yaml@main
    with:
      shellcheck: false
```

The above example disables the shellcheck linter; the other linters remain enabled.  All linters are enabled by default but can be selectively disabled if needed.

### reusable_purge_workflow_logs.yaml

A workflow log cleanup utility that automatically deletes old workflow runs to help manage repository storage and maintain a clean workflow history.

#### Inputs

| Input | Type | Default | Description |
|-------|------|---------|-------------|
| `retain-days` | number | `7` | Delete workflow runs older than this many days |

#### Permissions Required

This workflow requires `actions: write` permission to delete workflow runs.

#### Limitations

The workflow processes up to 1000 workflow runs per execution. For repositories with a large number of workflow runs exceeding this limit, run the workflow multiple times or schedule it to run more frequently.

#### Usage Example

To use this workflow in another repository, add the following to your `.github/workflows/purge-logs.yaml`:

```yaml
name: purge workflow logs

on:
  schedule:
    - cron: '0 0 * * *' # run daily at midnight UTC
  workflow_dispatch:

permissions:
  actions: write

jobs:
  purge-logs:
    uses: ulfiac/commons/.github/workflows/reusable_purge_workflow_logs.yaml@main
    with:
      retain-days: 7
```

This example runs the purge workflow daily and retains logs for 7 days.

### reusable_terraform_docs.yaml

Automatically generates and commits Terraform module documentation using [terraform-docs](https://terraform-docs.io/). When triggered, it scans a directory for Terraform modules and pushes updated `README.md` files directly to the branch.

#### Inputs

| Input | Type | Default | Description |
|-------|------|---------|-------------|
| `find-dir` | string | `terraform/modules` | Root directory to recursively search for Terraform modules |

#### Permissions Required

This workflow requires `contents: write` and `pull-requests: write` permissions to commit generated documentation.

#### Usage Example

To use this workflow in another repository, add the following to your `.github/workflows/terraform-docs.yaml`:

```yaml
name: terraform docs

on:
  push:
    branches:
      - main
    paths:
      - 'terraform/modules/**'
  workflow_dispatch:

permissions:
  contents: write
  pull-requests: write

jobs:
  terraform-docs:
    uses: ulfiac/commons/.github/workflows/reusable_terraform_docs.yaml@main
    with:
      find-dir: terraform/modules
```

## Composite Actions

### dump_context

A composite action for debugging reusable workflows. Dumps GitHub Actions context objects (env, github, inputs, job, matrix, runner, steps, strategy, vars) as grouped log output to help diagnose workflow issues.

#### Inputs

| Input | Type | Default | Description |
|-------|------|---------|-------------|
| `annotate` | boolean | `false` | Add a notice annotation to the workflow summary |
| `env` | boolean | `true` | Dump the `env` context |
| `github` | boolean | `true` | Dump the `github` context |
| `context_inputs` | boolean | `true` | Dump the `inputs` context |
| `job` | boolean | `true` | Dump the `job` context |
| `matrix` | boolean | `true` | Dump the `matrix` context |
| `runner` | boolean | `true` | Dump the `runner` context |
| `secrets` | boolean | `false` | Dump the `secrets` context (disabled by default to avoid exposure) |
| `steps` | boolean | `true` | Dump the `steps` context |
| `strategy` | boolean | `true` | Dump the `strategy` context |
| `vars` | boolean | `true` | Dump the `vars` context |

#### Usage Example

```yaml
steps:
  - uses: ulfiac/commons/.github/actions/dump_context@main
    with:
      secrets: false
```

## Contributing

When adding new reusable workflows:
- Ensure workflows are generic and configurable through inputs
- Use pinned action versions for security and reproducibility
- Test changes in dependent repositories before merging
- Update this README with new workflow documentation
- Update the copilot instructions

## Security

- Workflows run with minimal required permissions by default (`contents: read`)
- The purge workflow requires `actions: write` permission to delete workflow runs
- All actions use pinned versions to prevent supply chain attacks
- Regular security audits and dependency updates are performed

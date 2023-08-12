# GitHub Org Management

This GitHub organization is managed by [Peribolos](https://github.com/uwu-tools/peribolos).

## Peribolos Config

This tree roughly illustrates the necessary folder structure and configurations necessary to run Peribolos:

```bash
$ tree org

org # Main folder for Peribolos configuration
├── README.md # This README :)
└── openclarity # Parent folder for OpenClarity org structure
    ├── org.yaml # Main Peribolos configuration file
    └── vmclarity # Example team-based folder
        └── teams.yaml # Example team-based configuration file
```

The main configuration file is [org.yaml](./openclarity/org.yaml), with official documentation available [here](https://docs.prow.k8s.io/docs/components/cli-tools/peribolos/#org-configuration). Foundational organization settings should be defined here, in addition to core membership attributes, such as organization members, administrators, and owners. New members should be added to appropriately-scoped lists in _case-insensitive_ alphabetical order.

This configuration is supported by supplementary, team-based configuration files stored in respectively-named subfolders. For instance, the VMClarity project has its configuration stored in a dedicated [vmclarity/teams.yaml](../org/openclarity/vmclarity/teams.yaml) file. Note that repository access is _not_ explicitly declared in these configurations.

When Peribolos runs, it scans the entire [org/](../org) directory tree for `.yaml` files, and can merge multiple configuration files together, if applicable.

### Sample Config

The following is an example of a `teams.yaml` configuration file representing the teams `api-client-admins` and `api-client-maintainers`:

```yaml
# org/openclarity/api-client/teams.yaml
---
teams:
  api-client-admins:
    description: Admins for the API Client
    maintainers:
      - justaugustus
      - lelia
    privacy: closed
  api-client-maintainers:
    description: Maintainers of the API Client
    maintainers:
      - sambetts-cisco
    members:
      - edwarnicke
    privacy: closed
```

## GitHub Workflows

The following high-level structure is roughly used to compose Peribolos in a continuous integration environment powered by [reusable GitHub workflows](https://docs.github.com/en/actions/using-workflows/reusing-workflows):

```bash
$ tree .github

.github # GitHub repository configuration
└── workflows # GitHub Actions workflow files
    ├── _lint.yml # Reusable GitHub workflow for linting YAML
    ├── _stale.yml # Reusable GitHub workflow for stale issues
    ├── _sync.yml # Reusable GitHub workflow for running Peribolos
    ├── dry-run.yml # Lints config and runs Peribolos in dry-run mode
    ├── dump-config.yml # Dumps existing Peribolos org config
    ├── pr-lint.yml # Lints fork PRs after initial approval
    ├── prod-run.yml # Runs Peribolos to sync org changes
    └── stale-check.yml # Checks the repository for stale issues
```

### `prod-run.yml`

Peribolos presently runs on a scheduled (hourly cron) and event-driven (merges to `main`) basis. This production sync job is configured in [prod-run.yml](../.github/workflows/prod-run.yml), which in turn inherits logic from reusable workflows [_lint.yml](../.github/workflows/_lint.yml) and [_sync.yml](../.github/workflows/_sync.yml).

### `pr-lint.yml`

When organization changes are proposed via Pull Request, the PR will be reviewed by a member of @openclarity/org-admins for adherence to Peribolos syntax as well as organization policy. If the PR is initially approved, a [pr-lint.yml](../.github/workflows/pr-lint.yml) workflow will be triggered to lint any YAML updates submitted by the PR contributor. Note that this workflow is intentionally triggered by PR approvals in order to avoid checking out untrusted code submitted by external forks.

### `dry-run.yml`

Assuming that a proposed PR is approved, passes linting, and is merged into the default target branch (`develop`), a [dry-run.yml](../.github/workflows/dry-run.yml) workflow will be triggered next. Similarly to the production sync job, this workflow borrows from reusable workflows [_lint.yml](../.github/workflows/_lint.yml) and [_sync.yml](../.github/workflows/_sync.yml).

Since this invocation of Peribolos does not actually execute any changes to the organization structure, the dry run sync runs in parallel with linting, and if successful, will trigger a branch synchronization event between `develop` and `main`. Ultimately, the configuration committed to `main` is what will determine (and reinforce via cron schedule) the organization's membership, team affiliations, and high-level org metadata.

### `dump-config.yml`

Lastly, the [dump-config.yml](../.github/workflows/dump-config.yml) workflow generates a current snapshot of the GitHub organization config, from Peribolos' perspective. The resulting config dump will be output as a single block of YAML. The workflow can be [run manually](https://docs.github.com/en/actions/using-workflows/manually-running-a-workflow) using the `workflow_dispatch` mechanism.

## Proposing Changes

Please file any additions, changes, or removals to the organization configuration as pull requests [from your fork](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/creating-a-pull-request-from-a-fork) of this repository.

In the future, organization membership and team management capabilities will become self-service using a standardized workflow.

### Testing Changes

Any organization config changes proposed via fork PR will be linted after initial approval, and following a successful merge to `develop`, will trigger Peribolos in dry-run (`confirm: false`) mode to preview the output and prevent destructive changes.

If the configuration file is misconstructed, or there are YAML syntax issues, these should be caught by a combination of the [pr-lint.yml](../.github/workflows/pr-lint.yml) and [dry-run.yml](../.github/workflows/dry-run.yml) workflows.

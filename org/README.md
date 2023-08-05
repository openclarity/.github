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
    ├── prod-run.yml # Runs Peribolos to sync org changes
    └── stale-check.yml # Checks the repository for stale issues
```

Peribolos presently runs on a scheduled (hourly) and event-driven (merges to `main`) basis. These organization sync jobs are configured in [sync-org.yml](../.github/workflows/sync-org.yml). When new changes are proposed to the current configuration files stored in [org/](../org), a separate [dry-run.yml](../.github/workflows/dry-run.yml) job will run to lint the YAML and run Peribolos in dry-run mode to prevent erroneous changes from being deployed.

Lastly, the [dump-config.yml](../.github/workflows/dump-config.yml) workflow generates a current snapshot of the GitHub organization config, from Peribolos' perspective. The resulting config dump will be output as a single block of YAML. The workflow can be [run manually](https://docs.github.com/en/actions/using-workflows/manually-running-a-workflow) using the `workflow_dispatch` mechanism.

## Updates

Please file any additions, changes, or removals to the organization configuration as pull requests [from your fork](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/creating-a-pull-request-from-a-fork) of this repository.

> 💡 In the future, organization membership and team management capabilities will become self-service using a standardized workflow.

### Testing

As mentioned above, any changes pushed to the [org/](../org) directory will automatically run Peribolos in dry-run (`confirm: false`) mode to help prevent accidental, destructive changes. If the configuration file is misconstructed, or there are YAML syntax issues, these should be caught by the [dry-run.yml](../.github/workflows/dry-run.yml) workflow.

### Deploying

Any changes to the organization configuration must first be approved and merged as a pull request to `main`. Upon merge, the final [sync-org.yml](../.github/workflows/sync-org.yml) worklow will run and make the necessary changes.

This job also runs on an hourly `cron` schedule to help prevent organizational changes from being made outside of this audited process.

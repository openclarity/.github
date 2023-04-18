# GitHub Org Management

This GitHub organization is managed by [Peribolos](https://github.com/uwu-tools/peribolos).

## Structure

The following high-level structure is roughly used to compose Peribolos with GitHub workflows:

```bash
$ tree
.
├── .github # GitHub repo config
│   └── workflows # GitHub workflow files
│       ├── dry-run.yml # Dry-run of Peribolos
│       ├── dump-config.yml # Dump current org config
│       └── sync-org.yml # Sync orgs with Peribolos
└── org # Peribolos config directory
    ├── README.md # This README :)
    └── openclarity # Parent folder for org structure
        └── org.yaml # Main Peribolos configuration file
```

### Peribolos Config

The main configuration file is [org.yaml](./openclarity/org.yaml), with official documentation found [here](https://docs.prow.k8s.io/docs/components/cli-tools/peribolos/#org-configuration). This configuration is supported by supplementary, team-based configuration files stored in respectively-named subfolders.

For instance, the VMClarity project has its configuration stored in a dedicated [teams.yaml](../org/openclarity/vmclarity/teams.yaml). When Peribolos runs, it scans the entire [org/](../org) directory tree for `.yaml` files, and can merge multiple config files together, if applicable.

### GitHub Workflows

Peribolos presently runs on a scheduled (hourly) and event-driven (merges to `main`) basis. These organization sync jobs are configured in [sync-org.yml](../.github/workflows/sync-org.yml). When new changes are proposed to the current configuration files stored in [org/](../org), a separate [dry-run.yml](../.github/workflows/dry-run.yml) job will run to prevent erroneous changes from being deployed.

Lastly, the [dump-config.yml](../.github/workflows/dump-config.yml) workflow exists to generate a current snapshot of the GitHub organization config, from Peribolos' perspective. The resulting config dump will be output as a single block of YAML.

## Updates

Please file any additions, changes, or removals to the organization configuration as pull requests. In the future, user membership and team creation capabilities may become self-service using a standardized workflow.

### Testing

As mentioned above, any changes pushed to the [org/](../org) directory will automatically run Peribolos in dry-run (`confirm: false`) mode to help prevent accidental, destructive changes. If the configuration file is misconstructed, or there are YAML syntax issues, these should be caught by the [dry-run.yml](../.github/workflows/dry-run.yml) workflow.

### Deploying

Any changes to the organization configuration must first be approved and merged as a pull request to `main`. Upon merge, the final [sync-org.yml](../.github/workflows/sync-org.yml) worklow will run and make the necessary changes.

This job also runs on an hourly `cron` schedule to help prevent organizational changes from being made outside of this audited process.

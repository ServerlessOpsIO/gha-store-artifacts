# Store Artifacts GitHub Action

Store job artifacts in GitHub.

This GitHub Action allows you to store artifacts generated during your workflow. It supports setting custom artifact names and retaining artifacts for a specified number of days.

_*NOTE: This workflow is opinionated and meets the needs of its author. It is provided publicly as a reference for others to use and modify as needed.*_

The `gha-store-artifacts` action performs the following tasks:
1. Checks if the GitHub environment variable `GITHUB_REPOSITORY_OWNER_PART_SLUG_URL` is set and fails if not.
2. Optionally uploads a SAM artifact if specified.
3. Sets the artifact name based on the provided override or generates a default name.
4. Uploads the job working directory as an artifact to GHA

## Inputs

- `artifact_name_override` (optional): Override the name of the artifact.
- `artifact_retention_days` (optional): Number of days to retain artifacts..
- `use_aws_sam` (optional): Boolean to determine if SAM artifact should be uploaded.
- `aws_account_region` (optional): AWS region to use for SAM packaging.

## Outputs

- `artifact_name`: The name of the artifact stored.

## Usage

To use this action, add the following step to your workflow:

```yaml
name: CI

on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Setup job workspace
        id: setup-workspace
        uses: ServerlessOpsIO/gha-setup-workspace@v1

      # Do job work here

      - name: Store Artifacts
        uses: ServerlessOpsIO/gha-store-artifacts@v1
        with:
          use_aws_sam: true
          artifact_name_override: '${{ steps.store-artifacts.outputs.artifact_name }}'
```

### Artifact naming

Both [ServerlessOpsIO/gha-setup-workspace](https://github.com/ServerlessOpsIO/gha-setup-workspace) and [ServerlessOpsIO/gha-store-artifacts](https://github.com/ServerlessOpsIO/gha-store-artifacts) use the same [utility action](https://github.com/ServerlessOpsIO/gha-artifact-name) to set an artifact name if `artifact_name_override` is not set. If your workflow sets _artifact_name_override_ for `ServerlessOpsIO/gha-setup-workspace` be sure to use that actions _artifact_name_ output to set _artifact_name_override_ for `ServerlessOpsIO/gha-store-artifacts`.

See the example below:

```yaml
- name: Setup job workspace
  id: setup-workspace
  uses: ServerlessOpsIO/gha-setup-workspace@v1
  with:
    artifact_name_override: 'my-artifact'


- name: Store Artifacts
  uses: ServerlessOpsIO/gha-store-artifacts@v1
  with:
    artifact_name_override: '${{ steps.setup-workspace.outputs.artifact_name }}'
```

## Contributing

Contributions are welcome! Please open an issue or submit a pull request for any changes.

## Contact

For any questions or support, please open an issue in this repository.

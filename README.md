# Store Artifacts GitHub Action

Store job artifacts in GitHub.

This GitHub Action allows you to store artifacts generated during your workflow. It supports setting custom artifact names and retaining artifacts for a specified number of days.

For more information on what happens when `use_aws_sam` is true see [ServerlessOpsIO/gha-package-aws-sam]([gha-deploy-aws-sam](https://github.com/ServerlessOpsIO/gha-deploy-aws-sam/actions))

_*NOTE: This workflow is opinionated and meets the needs of its author. It is provided publicly as a reference for others to use and modify as needed.*_

The `gha-store-artifacts` action performs the following tasks:
1. Checks if the GitHub environment variable `GITHUB_REPOSITORY_OWNER_PART_SLUG_URL` is set and fails if not.
2. Optionally uploads a SAM artifact if specified.
3. Sets the artifact name based on the provided override or generates a default name.
4. Uploads the job working directory as an artifact to GHA

## Usage
See below for inputs, outputs, and examples.

### Inputs

- `artifact_name_override` (optional): Override the name of the artifact.
- `artifact_retention_days` (optional): Number of days to retain artifacts..
- `use_aws_sam` (optional): Boolean to determine if SAM artifact should be uploaded.
- `aws_account_region` (optional): AWS region to use for SAM packaging.
- `sam_s3_bucket` (optional): S3 bucket for SAM deployment.
- `sam_s3_prefix` (optional): S3 prefix for SAM deployment.

### Outputs

- `artifact-name`: The name of the artifact stored.
- `sam-artifact-bucket`: The S3 bucket where the SAM artifact was uploaded.
- `sam-artifact-bucket-prefix`: The S3 prefix where the SAM artifact was uploaded.

### Examples
To use this action see the examples below:

#### w/o AWS SAM
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
```

#### w/ AWS SAM
```yaml
name: CI
on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest

    # Required for AWS credentials
    permissions:
      id-token: write
      contents: read

    steps:
      - name: Setup job workspace
        id: setup-workspace
        uses: ServerlessOpsIO/gha-setup-workspace@v1


      # Do job work here


      # NOTE: ERunning `sam build` prior ensures Lambda function dependencies will be properly
      # bundled in the artifact.
      - name: SAM build
        id: sam-build
        shell: bash
        run: sam build

      - name: Assume AWS Credentials
        uses: ServerlessOpsIO/gha-assume-aws-credentials@v1
        with:
          build_aws_account_id: ${{ secrets.BUILD_AWS_ACCOUNT_ID }}
          deploy_aws_account_id: ${{ secrets.DEPLOY_AWS_ACCOUNT_ID }}
          aws_account_region: 'us-east-1'
          gha_build_role_name: ${{ secrets.GHA_BUILD_ROLE_NAME }}
          gha_deploy_role_name: ${{ secrets.GHA_DEPLOY_ROLE_NAME }}

      - name: Store Artifacts
        uses: ServerlessOpsIO/gha-store-artifacts@v1
        with:
          use_aws_sam: true

```

#### Artifact naming
Both [ServerlessOpsIO/gha-setup-workspace](https://github.com/ServerlessOpsIO/gha-setup-workspace) and [ServerlessOpsIO/gha-store-artifacts](https://github.com/ServerlessOpsIO/gha-store-artifacts) use the same [utility action](https://github.com/ServerlessOpsIO/gha-artifact-name) to set an artifact name if `artifact_name_override` is not set. If your workflow sets _artifact_name_override_ for `ServerlessOpsIO/gha-setup-workspace` be sure to use that actions _artifact_name_ output to set _artifact_name_override_ for `ServerlessOpsIO/gha-store-artifacts`.

See the example below:

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Setup job workspace
        uses: ServerlessOpsIO/gha-setup-workspace@v1
      - name: Store Artifacts
        uses: ServerlessOpsIO/gha-store-artifacts@v1
        with:
          artifact_name_override: 'MyArtifactName'
    outputs:
      artifact-name: ${{ steps.setup-workspace.outputs.artifact-name }}

  deploy:
    runs-on: ubuntu-latest
    needs:
      - build
    steps:
      - name: Setup job workspace
        uses: ServerlessOpsIO/gha-setup-workspace@v1
        with:
          checkout_artifact: true
          artifact_name_override: ${{ needs.build.outputs.artifact-name }}

```

#### S3 Bucket

The default value for `sam_s3_bucket` comes from [ServerlessOpsIO/aws-gha-integration](https://github.com/ServerlessOpsIO/aws-gha-integration). Both the `sam_s3_bucket` and `sam_s3_prefix` should not need to be configured but are avavailbel for unique circumstances.

## Contributing

Contributions are welcome! Please open an issue or submit a pull request for any changes.

## Contact

For any questions or support, please open an issue in this repository.

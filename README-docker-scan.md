# `gha-security/docker-scan`

## Usage

Add the following step to your workflow configuration:

```yml
jobs:
  docker-scan:
    name: Docker Scan
    uses: entur/gha-security/.github/workflows/docker-scan.yml@v0.2.0
    with:
        image_artifact: # The name of the image artifact to scan
    
```

## Inputs

<!-- AUTO-DOC-INPUT:START - Do not remove or modify this section -->

|                                   INPUT                                    |  TYPE  | REQUIRED | DEFAULT |      DESCRIPTION       |
|----------------------------------------------------------------------------|--------|----------|---------|------------------------|
| <a name="input_image_artifact"></a>[image_artifact](#input_image_artifact) | string |   true   |         | Image artifact to scan |

<!-- AUTO-DOC-INPUT:END -->

## Golden Path

- Docker image must be built before being scanned, preferably using reusable workflow `entur/gha-docker/.github/workflows/build.yml@v1`.

### Example

Let's look at an example, assume our repo is called `amazing-app`:

```sh
λ amazing-app ❯ tree
.
├── README.md
├── Dockerfile
└── .github
    └── workflows
        └── ci.yml
```

```yaml
# ci.yml
name: CI

on:
  pull_request:

jobs:
  docker-lint:
    uses: entur/gha-docker/.github/workflows/lint.yml@v1

  docker-build:
    uses: entur/gha-docker/.github/workflows/build.yml@v1

  docker-scan:
    needs: docker-build
    uses: entur/gha-security/.github/workflows/docker-scan.yml@v0.2.0
    with:
        image_artifact: ${{ needs.docker-build.outputs.image_artifact }}

  docker-push:
    uses: entur/gha-docker/.github/workflows/push.yml@v1
    secrets: inherit
```


## Allowlisting vulnerabilities
The reusable workflow uses the [Grype scanner](https://github.com/marketplace/actions/anchore-container-scan) to scan the Docker image for vulnerabilities. Any discovered vulnerabilities will be published to the _Security_ tab of the repository, under the _Code Scanning_ section. If you believe that a found vulnerability is a false positive or otherwise not relevant, you can either manually dimiss the alert, or create a allowlist file (YAML-file) that dismisses all alerts that matches a vulnerability ID. 

_NB! If the scan is performed on a pull request, remember to filter the Code Scanning results by pull request number and not the branch name._

There are two types of allowlist files that can be used; a local allowlist (placed in the repository) and an external allowlist (placed in a different repository). The local allowlist is prioritized over the external allowlist.

### Local allowlisting
To use a local allowlist, create a YAML file in the repository named, either `docker_scan_config.yml` or `docker_scan_config.yaml`, and place it in the root of the repository. If you create both files, the `docker_scan_config.yaml` file will be ignored. The file must follow the format [mentioned below](#schema-for-allowlist-file).

### External allowlisting
To use an external allowlist create a YAML file in a different repository and place it in the root of the repository. The file must be named either `docker_scan_config.yml` or `docker_scan_config.yaml`, and follow the format [mentioned below](#schema-for-allowlist-file). If you create both files, the `docker_scan_config.yaml` file will be ignored. To be able to use the external allowlist, you must have a local allowlist file in the repository as well. The repository where the external allowlist file is placed must be referenced in the local allowlist file, under the 'inherit' field. It is important to note that you will also have to use a fine-grained access token to access the external allowlist file. The token must be added as a secret to the repository where the workflow is run, and permissions must include "contents" read access to the repository where the external allowlist file is placed. You can find documentation on how to create a fine-grained access token [here](https://docs.github.com/en/enterprise-cloud@latest/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens#creating-a-fine-grained-personal-access-token), and how to add it as a secret to your repository [here](https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions#creating-secrets-for-a-repository). See the example below on how to use a repository secret, named `EXTERNAL_REPOSITORY_TOKEN`, in the workflow.

```yaml
jobs:
  docker-scan:
    needs: docker-build
    uses: entur/gha-security/.github/workflows/docker-scan.yml@v0.2.0
    with:
        image_artifact: ${{ needs.docker-build.outputs.image_artifact }}
    secrets:
        external_repository_token: ${{ secrets.EXTERNAL_REPOSITORY_TOKEN }}
```

### Schema for whitelist file
```yaml
apiVersion: entur.io/v1alpha1
kind: DockerScanConfig
metadata:
  name: {project-name-config}
spec:
  inherit: {repository where the external allowlist file is placed}
  whitelist:
  - cve: {cve-id}
    comment: {comment explaining why the vulnerability is dismissed}
    reason: {reason for dismissing the vulnerability}
```
The `name` field under the metadata section should be the name of the project. The `inherit` field under the spec section should be the repository where the external allowlist file is placed (optinal field, only used when using an external allowlist file). The `allowlist` field should be a list of vulnerabilities that you want to dismiss. For each vulnerability you want to dismiss, you should add a new item to the list. For each item in the list, you should add the following fields: `cve`, `comment`, and `reason`. The `cve` field should be the CVE-ID of the vulnerability you want to dismiss, the `comment` field should be a comment explaining why the vulnerability is dismissed, and the `reason` field should be a short description on why the vulnerability is dismissed.

#### Example

```yaml
apiVersion: entur.io/v1alpha1
kind: DockerScanConfig
metadata:
  name: my-project-config
spec:
  whitelist:
  - cve: "CVE-2021-1234-abc"
    comment: "This alert is a false positive"
    reason: "false_positive"
```

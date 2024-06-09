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

|                                           INPUT                                           |  TYPE  | REQUIRED |    DEFAULT     |                 DESCRIPTION                  |
|-------------------------------------------------------------------------------------------|--------|----------|----------------|----------------------------------------------|
| <a name="input_image_artifact"></a>[image_artifact](#input_image_artifact)                | string |  true    |                |  The name of the image artifact to scan      |
| <a name="input_image_whitelist"></a>[image_whitelist](#input_image_whitelist) | string |  false   | `"image_whitelist.yaml"` | The path to the file <br>containing the whitelisting rules, starting <br>from the root of the <br>repository  |

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

  docker-image-scan:
    needs: docker-build
    uses: entur/gha-security/.github/workflows/docker-scan.yml@v0.2.0
    with:
        image_artifact: ${{ needs.docker-build.outputs.image_artifact }}

  docker-push:
    uses: entur/gha-docker/.github/workflows/push.yml@v1
    secrets: inherit
```


## Whitelisting vulnerabilities
The reusable workflow uses the [Grype scanner](https://github.com/marketplace/actions/anchore-container-scan) to scan the Docker image for vulnerabilities. Any discovered vulnerabilities will be published to the _Security_ tab of the repository, under the _Code Scanning_ section. If you believe that a found vulnerability is a false positive or otherwise not relevant, you can either manually dimiss the alert, or create whitelist files (YAML-file) that dismisses all alerts that matches a vulnerability ID.

NB! If the scan is performed on a pull request, remember to filter by pull request number and not the branch name. 

There are two types of whitelisting files that can be used; a local file (placed in the repository) and an external file (placed in a different repository). The local file is prioritized over the external file.

### Local whitelisting
To use local whitelisting, create a file in the repository and place it preferably in the root of the repository. The file must be a YAML-file and follow the format [mentioned below](#schema-for-whitelist-file).

#### Example of workflow using local whitelisting
```yaml
jobs:
  docker-scan:
    name: Docker Scan
    uses: entur/gha-security/.github/workflows/docker-scan.yml@v0.2.0
    with:
        image_artifact: ${{ needs.docker-build.outputs.image_artifact }}
        local_whitelist: "image_whitelist.yaml"
```

### External whitelisting
To use external whitelisting, create a file in a different repository and place it preferably in the root of the repository. The file should be named `image_whitelist.yaml` and follow the format [mentioned below](#schema-for-whitelist-file). It is important to note that if the repository where the whitelist file is placed is private or internal, you must create a fine-grained access token with permission set "Contents" repository permissions (read) and add it as a secret in the repository where the scan is performed. 
You can find documentation on how to create a fine-grained access token [here](https://docs.github.com/en/enterprise-cloud@latest/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens#creating-a-fine-grained-personal-access-token), and how to add it as a secret to your repository [here](https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions#creating-secrets-for-a-repository). 

#### Example of workflow using external whitelisting
```yaml
jobs:
  docker-scan:
    name: Docker Scan
    uses: entur/gha-security/.github/workflows/docker-scan.yml@v0.2.0
    with:
        image_artifact: ${{ needs.docker-build.outputs.image_artifact }}
        external_whitelist: "https://raw.githubusercontent.com/entur/whitelist-repo/main/image_whitelist.yaml"
        external_repository: "external-repo"
    secrets:
        external_repository_token: ${{ secrets.EXTERNAL_REPO_TOKEN }}
```


### Schema for whitelist file
```yaml
apiVersion: entur.io/v1alpha1
kind: DockerScanConfig
metadata:
  name: {project-name-config}
spec:
  whitelist:
  - cve: {cve-id}
    comment: {comment explaining why the vulnerability is dismissed}
    reason: {reason for dismissing the vulnerability}
```
The metadata field should be the name of the project, the `cve` field should be the CVE-ID of the vulnerability you want to dismiss, the `comment` field should be a comment explaining why the vulnerability is dismissed, and the `reason` field should be a short description on why the vulnerability is dismissed.

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

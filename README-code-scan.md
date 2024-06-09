# `gha-security/code-scan`

## Usage

Add the following step to your workflow configuration:

```yml
jobs:
  code-scan:
    name: Code Scan
    uses: entur/gha-security/.github/workflows/code-scan.yml@v0.2.0
```

## Inputs

<!-- AUTO-DOC-INPUT:START - Do not remove or modify this section -->

|                                           INPUT                                           |  TYPE  | REQUIRED | DEFAULT |                   DESCRIPTION                   |
|-------------------------------------------------------------------------------------------|--------|----------|---------|-------------------------------------------------|
| <a name="input_code_whitelist"></a>[code_whitelist](#input_code_whitelist) | string |  false   |         | Whitelisting file for code scanning <br>alerts  |

<!-- AUTO-DOC-INPUT:END -->

## Golden Path

- Workflow must be named `codeql.yml`.

### Example

```yaml
# codeql.yml
name: "CodeQL"

on:
    pull_request:
        branches:
            - main
            - master
    push:
        branches:
            - main
            - master
  
jobs:
    code-scan:
        name: Code Scan
        uses: entur/gha-security/.github/workflows/code-scan.yml@v0.2.0
```

## Whitelisting vulnerabilities
The reusable workflow uses [CodeQL](https://codeql.github.com/) to scan the codebase for vulnerabilities.  Any discovered vulnerabilities will be published to the _Security_ tab of the repository, under the _Code Scanning_ section. If you believe that a found vulnerability is a false positive or otherwise not relevant, you can either manually dimiss the alert, or create whitelist files (YAML-file) that dismisses all alerts that matches a vulnerability ID.

NB! If the scan is performed on a pull request, remember to filter by pull request number and not the branch name. 

There are two types of whitelisting files that can be used; a local file (placed in the repository) and an external file (placed in a different repository). The local file is prioritized over the external file.

### Local whitelisting
To use local whitelisting, create a file in the repository and place it preferably in the root of the repository. The file must be a YAML-file and follow the format [mentioned below](#schema-for-whitelist-file).

#### Example of workflow using local whitelisting
```yaml
jobs:
  code-scan:
    name: Code Scan
    uses: entur/gha-security/.github/workflows/code-scan.yml@v0.2.0
    with:
      local_whitelist: "code_whitelist.yaml"
```

### External whitelisting
To use external whitelisting, create a file in a different repository and place it preferably in the root of the repository. The file must be a YAML-file and follow the format [mentioned below](#schema-for-whitelist-file). It is important to note that if the repository where the whitelist file is placed is private or internal, you must create a fine-grained access token with permission set "Contents" repository permissions (read) and add it as a secret in the repository where the scan is performed. 
You can find documentation on how to create a fine-grained access token [here](https://docs.github.com/en/enterprise-cloud@latest/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens#creating-a-fine-grained-personal-access-token), and how to add it as a secret to your repository [here](https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions#creating-secrets-for-a-repository). 

#### Example of workflow using external whitelisting
```yaml
jobs:
  code-scan:
    name: Code Scan
    uses: entur/gha-security/.github/workflows/code-scan.yml@v0.2.0
    with:
      external_whitelist: "https://raw.githubusercontent.com/entur/whitelist-repo/main/code_whitelist.yaml"
      external_repository: "external-repo"
    secrets:
      external_repository_token: ${{ secrets.EXTERNAL_REPOSITORY_TOKEN }}
```


### Schema for whitelist file
```yaml
apiVersion: entur.io/v1alpha1
kind: CodeScanConfig
metadata:
  name: {project-name-config}
spec:
  whitelist:
  - cwe: {cwe-id}
    comment: {comment explaining why the vulnerability is dismissed}
    reason: {reason for dismissing the vulnerability}
```
The metadata field should be the name of the project, the `cwe` field should be the CWE-ID of the vulnerability you want to dismiss, the `comment` field should be a comment explaining why the vulnerability is dismissed, and the `reason` field should be one of the following:
- "false_positive"
- "wont_fix"
- "test"

#### Example

```yaml
apiVersion: entur.io/v1alpha1
kind: CodeScanConfig
metadata:
  name: my-project-config
spec:
  whitelist:
  - cwe: "cwe-080"
    comment: "This alert is a false positive"
    reason: "false_positive"
  - cwe: "cwe-916"
    comment: "Wont be able to fix this in the near future"
    reason: "wont_fix"
  - cwe: "cwe-400"
    comment: "Used for testing purposes"
    reason: "test"  
```
# `gha-security/code_scanning`

## Usage

Add the following step to your workflow configuration:

```yml
jobs:
  code-scanning:
    name: Code Scanning
    uses: entur/gha-security/.github/workflows/code_scanning.yml@main
```

## Inputs

<!-- AUTO-DOC-INPUT:START - Do not remove or modify this section -->

|                                   INPUT                                    |  TYPE  | REQUIRED |          DEFAULT           |                                                  DESCRIPTION                                                  |
|----------------------------------------------------------------------------|--------|----------|----------------------------|---------------------------------------------------------------------------------------------------------------|
| <a name="input_whitelist-file"></a>[whitelist-file](#input_whitelist-file) | string |  false   | `"code_whitelisting.yaml"` | The path to the file <br>containing the whitelisting rules, starting <br>from the root of the <br>repository  |

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
  
jobs:
    code-scanning:
        name: Code Scanning
        uses: entur/gha-security/.github/workflows/code_scanning.yml@main
```

### White-listing vulnerabilities
The reusable workflow uses [CodeQL](https://codeql.github.com/) to scan the codebase for vulnerabilities. The scanner will fail if any critical vulnerabilities are found. If you believe that a found vulnerbility is a false positive or otherwise not relevant, you can either manually dimiss the alert, or create a whitelist file (YAML-file) that dismisses all alerts that matches a vulnerability ID.

The whitelist file should be placed in the root of the repository, and the path to the file should be provided as an input to the workflow. The file should have the following format:

```yaml
- cwe: external/cwe/{cwe-id}
  dismissed_reason: false positive / won't fix / used in tests 
  dismissed_comment: {reason for dismissal}
```

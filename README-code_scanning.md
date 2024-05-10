# `gha-security/code_scanning`

## Usage

Add the following step to your workflow configuration:

```yml
jobs:
  code-scanning:
    name: Code Scanning
    uses: entur/gha-security/.github/workflows/code_scanning.yml@main
```

## Golden Path

- Workflow must be named `codeql.yml`.

### Example

```yaml
# codeql.yml
```
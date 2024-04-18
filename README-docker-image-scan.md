# `gha-security/docker-image-scan`

## Usage

Add the following step to your workflow configuration:

```yml
jobs:
  docker-image-scan:
    name: Docker Image Scan
    uses: entur/gha-security/.github/workflows/docker_image_scan.yml@main
    with:
        image_artifact: # The name of the image artifact to scan
```

## Inputs

<!-- AUTO-DOC-INPUT:START - Do not remove or modify this section -->

|                                           INPUT                                           |  TYPE  | REQUIRED |    DEFAULT     |                 DESCRIPTION                  |
|-------------------------------------------------------------------------------------------|--------|----------|----------------|----------------------------------------------|
| <a name="input_image_artifact"></a>[image_artifact](#input_image_artifact)                | string |  true    |                |  The name of the image artifact to scan      |

<!-- AUTO-DOC-INPUT:END -->

## Golden Path

- Docker image must be built before being scanned, preferably using reusable workflow `entur/gha-docker/.github/workflows/build.yml@main`.

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
    uses: entur/gha-docker/.github/workflows/lint.yml@main

  docker-build:
    uses: entur/gha-docker/.github/workflows/build.yml@main

  docker-image-scan:
    needs: docker-build
    uses: entur/gha-security/.github/workflows/docker-image-scan.yml@main
    with:
        image_artifact: ${{ needs.docker-build.outputs.image_artifact }}

  docker-push:
    needs: docker-image-scan
    uses: entur/gha-docker/.github/workflows/push.yml@main
    secrets: inherit
```


### White-listing vulnerabilities
The reusable workflow uses the [Grype scanner](https://github.com/marketplace/actions/anchore-container-scan) to scan the Docker image for vulnerabilities. The scanner will fail if any critical vulnerabilities are found. If you believe that a found vulnerbility is a false positive or otherwise not relevant, you can tell Grype to ignore matches by specifying one or more "ignore rules" in your Grype configuration file (add a .grype.yaml file at your repository root). This causes Grype not to report any vulnerability matches that meet the criteria specified by any of your ignore rules.

Each rule can specify any combination of the following criteria:
- vulnerability ID (e.g. `"CVE-2008-4318"`)
- namespace (e.g. `"nvd"`)
- fix state (allowed values: `"fixed"`, `"not-fixed"`, `"wont-fix"`, or `"unknown"`)
- package name (e.g. `"libcurl"`)
- package version (e.g. `"1.5.1"`)
- package language (e.g. `"python"`; these values are defined [here](https://github.com/anchore/syft/blob/main/syft/pkg/language.go#L14-L23))
- package type (e.g. `"npm"`; these values are defined [here](https://github.com/anchore/syft/blob/main/syft/pkg/type.go#L10-L24))
- package location (e.g. `"/usr/local/lib/node_modules/**"`; supports glob patterns)


Here's an example `~/.grype.yaml` that demonstrates the expected format for ignore rules:

```yml	
ignore:
  # This is the full set of supported rule fields:
  - vulnerability: CVE-2008-4318
    fix-state: unknown
    # VEX fields apply when Grype reads vex data:
    vex-status: not_affected
    vex-justification: vulnerable_code_not_present
    package:
      name: libcurl
      version: 1.5.1
      type: npm
      location: "/usr/local/lib/node_modules/**"

  # We can make rules to match just by vulnerability ID:
  - vulnerability: CVE-2014-54321

  # ...or just by a single package field:
  - package:
      type: gem
```


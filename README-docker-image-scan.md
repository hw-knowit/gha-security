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
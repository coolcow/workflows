# coolcow/workflows

This repository centralizes CI/CD workflows, promoting consistency and reducing duplication across various projects within the coolcow repositories.

## Purpose

The primary goal of this repository is to provide a single source of truth for common GitHub Actions workflows used across the `coolcow` organization. By centralizing these workflows, we aim to:

*   **Reduce Duplication:** Avoid copying and pasting workflow logic into every repository.
*   **Improve Consistency:** Ensure all projects follow standardized CI/CD practices.
*   **Simplify Maintenance:** Update a workflow in one place, and all consuming repositories automatically benefit from the changes.
*   **Accelerate Development:** Quickly set up CI/CD for new projects by simply calling a reusable workflow.

## Usage

To use a reusable workflow from this repository in one of your `coolcow` projects, you need to create a workflow file (e.g., `.github/workflows/build-and-publish.yml`) in your project and call the desired workflow.

**Example: Calling the `build-and-publish.yml` workflow**

```yaml
# .github/workflows/your-project-ci.yml (in your project repository)
name: Your Project CI

on:
  push:
    tags: [ 'v*.*.*' ] # Or your desired trigger

jobs:
  build-and-publish:
    uses: coolcow/workflows/.github/workflows/build-and-publish.yml@main # Replace 'main' with a specific tag or commit SHA for stability
    with:
      image_name: 'your-project-image-name' # Required: The specific name for your project's Docker image
      # build_context: '.' # Optional: If your Dockerfile is not at the root of the context, specify it here
      # test_dockerfile_path: 'build/Dockerfile.test' # Optional: Path to a Dockerfile for testing asset images
    secrets: inherit # Required: Allows the reusable workflow to access secrets from your project
```

### Inputs

The `build-and-publish.yml` workflow accepts the following inputs:

| Input                  | Description                                                                                                                                                            | Required | Default            |
| ---------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- | ------------------ |
| `image_name`           | The specific name for the project's Docker image, which will be prefixed with `coolcow/`.                                                                                | `true`   | `N/A`              |
| `dockerfile_path`      | The path to the main `Dockerfile`.                                                                                                                                     | `false`  | `build/Dockerfile` |
| `build_context`        | The build context path for the Docker build.                                                                                                                           | `false`  | `.`                |
| `test_dockerfile_path` | Optional path to a `Dockerfile` used for testing. If provided, the workflow runs an intermediate test step. This is ideal for verifying the contents of asset images. | `false`  | `''`               |

### Asset Image Testing

The optional `test_dockerfile_path` input enables a two-step build process ideal for "asset images" (e.g., `FROM scratch` images containing only scripts or files).

1.  **Build Local:** The primary image is built and loaded into the runner's local Docker daemon with a temporary tag.
2.  **Test:** A second build is initiated using the `test_dockerfile_path`. This test `Dockerfile` should use the temporary image as its base (via a build argument `ASSET_IMAGE`) and run commands to verify its contents. If any `RUN` command fails, the entire workflow fails.

**Example `Dockerfile.test`:**
```dockerfile
# The asset image to test is passed via build-arg
ARG ASSET_IMAGE
FROM ${ASSET_IMAGE} AS assets_to_test

# Use a standard image with a shell for testing
FROM alpine:latest

# Copy the contents from the asset image to a testable location
COPY --from=assets_to_test /assets/ /testdata/

# Run verification commands. A failure here will fail the build.
RUN test -f /testdata/my_script.sh && \
    grep -q "expected content" /testdata/my_script.sh
```

**Important Notes:**

*   Always specify a **specific tag or commit SHA** for the `uses:` reference (e.g., `@v1.0.0` or `@abcdef123`) in production environments for stability, rather than `@main`.
*   Ensure that the calling repository has appropriate permissions for `contents: read` and `packages: write` if the reusable workflow performs these actions.

## Contributing

Contributions are welcome! If you need a new reusable workflow or want to improve an existing one, please follow these steps:

1.  Fork this repository.
2.  Create a new branch for your feature or fix.
3.  Implement your changes, including tests if applicable.
4.  Update the documentation (`README.md`) if your changes affect usage.
5.  Open a pull request describing your changes and their benefits.

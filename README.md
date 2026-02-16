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
    secrets: inherit # Required: Allows the reusable workflow to access secrets from your project
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

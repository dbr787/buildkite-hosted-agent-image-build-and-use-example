# Buildkite Hosted Agent Image Build and Use Example

[![Add to Buildkite](https://buildkite.com/button.svg)](https://buildkite.com/new)

This example shows how to build a custom container image in one pipeline step, and use it as the base agent image in which the next step will run with Buildkite hosted agents.

## What This Does

The pipeline uses the `image` attribute to automatically use your custom built image for all steps.

**Step 1: Create Custom Base Image**

- Runs on the default queue image (specified by `image: ~`) to avoid recursion
- Builds a minimal Ubuntu image with the current build number embedded
- Pushes to `${BUILDKITE_HOSTED_REGISTRY_URL}/base:latest` (your workspace's internal registry)
- Only rebuilds when `.buildkite/Dockerfile.build` or `.buildkite/pipeline.yml` changes

**Step 2: Use Custom Base Image**

- Runs in parallel (3 instances) to demonstrate the custom image works across multiple jobs
- Automatically uses the custom image via the pipeline-level `image` attribute
- Verifies it's using the correct custom image by displaying the build number marker

## How It Works

The pipeline sets a default image at the top level:

```yaml
image: "${BUILDKITE_HOSTED_REGISTRY_URL}/base:latest"
```

This means all steps automatically use your custom image unless they specify a different one. The `BUILDKITE_HOSTED_REGISTRY_URL` environment variable is automatically provided by Buildkite and points to your workspace's internal container registry.

## Files

- `.buildkite/pipeline.yml` - Pipeline with image attribute and two steps
- `.buildkite/Dockerfile.build` - Simple Ubuntu base with build number marker

## Setup Instructions

**Important**: This example requires Buildkite Hosted Agents. It will not work with self-hosted agents as the `BUILDKITE_HOSTED_REGISTRY_URL` environment variable is only available in hosted environments.

**Note**: The `image` attribute is currently experimental. See the [container image attributes documentation](https://buildkite.com/docs/pipelines/configure/step-types/command-step#container-image-attributes) for more details.

**Warning**: If you build or use a broken image, error messages and logging may be limited. Ensure your Dockerfile builds successfully and the resulting image is valid before relying on it in your pipeline.

1. **Create Pipeline**: Connect this repo to a new Buildkite pipeline using the "Add to Buildkite" button above.

2. **Run the Pipeline**: That's it! The pipeline will:
   - Build your custom image using the public base image
   - Automatically use the built image in subsequent steps
   - Show the build number marker proving the custom image is in use

# Buildkite Hosted Agent Image Build and Use Example

[![Add to Buildkite](https://buildkite.com/button.svg)](https://buildkite.com/new)

This example shows how to build a custom container image in one pipeline step, and use it as the base agent image in which the next step will run with Buildkite hosted agents.

## What This Does

The pipeline demonstrates how to build a custom Docker image at the start of a pipeline, and then use that image as the execution environment for subsequent steps.

It uses the [`image` attribute](https://buildkite.com/docs/pipelines/configure/step-types/command-step#container-image-attributes) to specify the custom image for the steps that need it. ()

**Step 1: Create Custom Base Image**

- Runs on the default queue image (configured in the queue settings)
- Builds a minimal Ubuntu image with the current build number embedded
- Pushes to `${BUILDKITE_HOSTED_REGISTRY_URL}/base:latest` (your workspace's internal registry)
- Only rebuilds when `.buildkite/Dockerfile.build` or `.buildkite/pipeline.yml` changes

**Step 2: Use Custom Base Image**

- Runs in parallel (3 instances) to demonstrate the custom image works across multiple jobs
- Uses the custom image via the step-level `image` attribute
- Verifies it's using the correct custom image by displaying the build number marker

## How It Works

The pipeline sets the image on the step that needs it:

```yaml
- key: use_custom_base_image
  image: "${BUILDKITE_HOSTED_REGISTRY_URL}/base:latest"
  # ...
```

This means you can mix standard steps (using the default queue image) and steps using your custom image in the same pipeline. The `BUILDKITE_HOSTED_REGISTRY_URL` environment variable is automatically provided by Buildkite and points to your workspace's internal container registry.

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

# Claude in Jupyter

A containerized Jupyter Data Science workbench with Google Cloud SDK integration, built for OpenShift AI (RHOAI).

## Overview

This project extends the Red Hat OpenShift AI Jupyter Data Science workbench image with Google Cloud CLI tools, enabling seamless interaction with GCP services from Jupyter notebooks.

## Components

### Container Image

- **Base Image**: Red Hat OpenShift AI Jupyter Data Science (Python 3.12, RHEL 9)
- **Additions**: Google Cloud SDK v562.0.0 and required dependencies
- **Location**: `claude_workbench/CONTAINERFILE`

### OpenShift Resources

**BuildConfig** (`claude_workbench/buildconfig.yaml`):
- Builds container image from Git repository
- Pushes to internal OpenShift registry
- ImageStream: `jupyter-datascience-claude`

**Tekton Pipeline** (`claude_workbench/pipeline.yaml`):
- Automated build pipeline with Git integration
- Tags images with commit SHA for version tracking
- Includes workspace PVC and ServiceAccount configuration

## Prerequisites

- OpenShift 4.x cluster with:
  - OpenShift Pipelines (Tekton) operator installed
  - Access to internal image registry
  - Permissions to create BuildConfigs, Pipelines, and ImageStreams

## Deployment

### 1. Apply BuildConfig and ImageStream

```bash
oc apply -f claude_workbench/buildconfig.yaml
```

### 2. Deploy Tekton Pipeline

```bash
# Apply pipeline definition and workspace
oc apply -f claude_workbench/pipeline.yaml

# Create ServiceAccount and start pipeline
oc create -f claude_workbench/pipeline-run.yaml
```

### 3. Manual Build (Alternative)

```bash
# Trigger build directly
oc start-build jupyter-datascience-claude --follow
```

## Pipeline Workflow

The Tekton pipeline automates the build process:

1. **git-clone**: Clones this repository
2. **get-commit-sha**: Extracts Git commit SHA
3. **trigger-build**: Starts OpenShift BuildConfig
4. **tag-image**: Tags resulting image with commit SHA

### Image Tags

Each successful pipeline run creates:
- `jupyter-datascience-claude:latest` - Most recent build
- `jupyter-datascience-claude:<sha>` - Specific commit version (e.g., `90638c9`)
- `jupyter-datascience-claude:git-<sha>` - Alternate commit tag (e.g., `git-90638c9`)

## Using the Image

Reference the image in your Jupyter notebook deployment:

```yaml
image: image-registry.openshift-image-registry.svc:5000/<namespace>/jupyter-datascience-claude:latest
```

Or use a specific version:

```yaml
image: image-registry.openshift-image-registry.svc:5000/<namespace>/jupyter-datascience-claude:90638c9
```

## Development

To modify the container image:

1. Edit `claude_workbench/CONTAINERFILE`
2. Commit and push changes
3. Run the pipeline or trigger a manual build

The BuildConfig automatically pulls from the `main` branch.

## Project Structure

```
.
├── CLAUDE.md                          # Claude Code instructions
├── README.md                          # This file
└── claude_workbench/
    ├── CONTAINERFILE                  # Container image definition
    ├── buildconfig.yaml               # OpenShift BuildConfig + ImageStream
    ├── pipeline.yaml                  # Tekton pipeline definition
    └── pipeline-run.yaml              # Pipeline ServiceAccount + PipelineRun
```

## License

This project is provided as-is for use with Red Hat OpenShift AI environments.

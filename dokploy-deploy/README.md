# Docker Build, Cache & Dokploy Deploy Action

This GitHub composite action builds a Docker image, pushes it to a container registry (like GitHub Container Registry - GHCR), caches it using GitHub Actions cache provider, and triggers a Dokploy deployment webhook with push event payload.

## Features

- ⚡ **Docker Buildx & Caching**: Speeds up subsequent builds using native `gha` (GitHub Actions) cache.
- 📦 **Multi-Registry Support**: Defaults to GitHub Container Registry (`ghcr.io`) but supports arbitrary registries.
- 🛡️ **Flexible Architecture**: Build for multiple target platforms (default: `linux/amd64`).
- 🚀 **Dokploy Webhook Emulation**: Automatically generates a GitHub-like push webhook payload (including ref, SHA, and commit message) to satisfy Dokploy's Git webhook integration format.

## Usage

Create a workflow file (e.g., `.github/workflows/deploy.yml`) in your repository:

```yaml
name: Deploy to Dokploy

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write

    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Deploy to Dokploy
        uses: fongfan999/github-actions/dokploy-deploy@main
        with:
          deploy_hook_url: ${{ secrets.DOKPLOY_DEPLOY_HOOK_URL }}
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `deploy_hook_url` | Dokploy deployment webhook URL | **Yes** | N/A |
| `context` | Docker build context | No | `.` |
| `file` | Path to the Dockerfile | No | `./Dockerfile` |
| `image_name` | Docker image name (defaults to repository name in lowercase) | No | `${GITHUB_REPOSITORY,,}` |
| `registry` | Docker registry | No | `ghcr.io` |
| `username` | Username for Docker registry | No | `${{ github.actor }}` |
| `password` | Password/Token for Docker registry | No | `${{ github.token }}` |
| `no_cache` | Disable Docker cache (`true` or `false`) | No | `false` |
| `platforms` | Target platforms for build | No | `linux/amd64` |
| `build_args` | List of custom build-time variables (one per line) | No | N/A |

## Webhook Payload Structure

This action triggers the Dokploy webhook by sending a `POST` request with the `X-GitHub-Event: push` header and a JSON payload containing:

```json
{
  "ref": "refs/heads/main",
  "head_commit": {
    "id": "1234567abcdef...",
    "message": "Commit message here"
  }
}
```

This mirrors a GitHub webhook payload, allowing Dokploy to pull and deploy the correct image.

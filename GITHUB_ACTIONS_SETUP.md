# GitHub Actions Setup for Discourse CI/CD

## Overview
This CI/CD pipeline automates Docker image builds, testing, and deployment for Discourse.

**Workflows included:**
- `docker-build-push.yml` — Builds and pushes Docker images to Docker Hub
- `tests.yml` — Runs automated tests on every push and PR

## Setup Instructions

### 1. Add Docker Hub Secrets to GitHub

Go to your repository **Settings → Secrets and variables → Actions** and add:

| Secret Name | Value |
|------------|-------|
| `DOCKERHUB_USERNAME` | Your Docker Hub username |
| `DOCKERHUB_TOKEN` | Your Docker Hub access token (create at hub.docker.com/settings/security) |

**How to create a Docker Hub token:**
1. Go to https://hub.docker.com/settings/security
2. Click "New Access Token"
3. Give it a name (e.g., "GitHub Actions")
4. Copy the token and paste it into GitHub Secrets as `DOCKERHUB_TOKEN`

### 2. Branch Configuration

The workflows trigger on:
- **Push** to `main`, `develop`, or version tags (`v*`)
- **Pull requests** against `main` or `develop`

Edit the `branches` and `tags` lists in the workflow files to match your branching strategy.

### 3. Image Tagging Strategy

The `docker-build-push.yml` workflow automatically tags images as:
- `<username>/discourse:branch-name` (on push to branches)
- `<username>/discourse:v1.0.0` (on release tags)
- `<username>/discourse:sha-abc123def` (git commit SHA)
- `<username>/discourse:latest` (on main branch)

### 4. Test Requirements

The `tests.yml` workflow assumes:
- Ruby project with RSpec tests
- PostgreSQL and Redis services
- Rubocop and ESLint linters
- Coverage reports in `./coverage/coverage.xml`

Adjust the commands if your project uses different test runners or languages.

### 5. Dockerfile Requirements

Ensure your repository includes a `Dockerfile` in the root directory. For Discourse, use a multi-stage build (see `Dockerfile.example`).

### 6. Running Locally

Test the Docker build locally before pushing:

```bash
docker buildx build -t discourse:test .
docker run --rm discourse:test bundle exec rake db:migrate
```

### 7. Monitoring Builds

After pushing to GitHub:
1. Go to **Actions** tab in your repository
2. View workflow runs and logs
3. Check "Build and Push Docker Image" for image push status
4. Check "Automated Testing" for test failures

### 8. Troubleshooting

**Build fails with "Image not found"**
- Ensure all base images (e.g., `ruby:3.2`) are publicly available
- Check Dockerfile COPY paths match your repository structure

**Push fails with "Invalid credentials"**
- Verify `DOCKERHUB_USERNAME` and `DOCKERHUB_TOKEN` are correctly set in GitHub Secrets
- Regenerate the token at hub.docker.com/settings/security if needed

**Tests fail in CI but pass locally**
- Ensure test environment variables in `tests.yml` match your local setup
- PostgreSQL/Redis services may have different connection strings in CI

**Dockerfile linting fails (Hadolint)**
- Review warnings at https://github.com/hadolint/hadolint/wiki
- Common issues: missing `LABELS`, non-root user recommendations, using `latest` tags

## Advanced: Build Cache

The workflows use GitHub Actions Cache to speed up builds:
- Images are cached in the GitHub Actions runner
- Cache is shared across runs on the same branch
- Cache expires after 7 days of inactivity

To force a cache rebuild, add a comment in your commit message or manually trigger a workflow run.

## Advanced: Multi-Registry Deployment

To push to multiple registries (Docker Hub + GitHub Container Registry):

```yaml
images: |
  docker.io/${{ secrets.DOCKERHUB_USERNAME }}/discourse
  ghcr.io/${{ github.repository }}
```

Add `GHCR_TOKEN` secret (GitHub token with write:packages permission) for GHCR.

## Next Steps

1. **Add branch protection rules** — Require status checks to pass before merging
2. **Add deployment workflows** — Auto-deploy to staging/production after successful builds
3. **Set up notifications** — Slack/email alerts for failed builds
4. **Monitor image size** — Add size checks to prevent bloated images

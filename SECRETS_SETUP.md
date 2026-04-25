# GitHub Actions Secrets & Variables Reference

## Required Secrets (must be added manually in GitHub UI)

### Docker Hub Authentication
- **DOCKERHUB_USERNAME**: Your Docker Hub account username
  - Create at: https://hub.docker.com/
  - Example: `john-doe`

- **DOCKERHUB_TOKEN**: Personal Access Token for authentication
  - Create at: https://hub.docker.com/settings/security
  - **DO NOT use your Docker Hub password** — use a token instead
  - Permissions: Read & Write
  - Example: `dckr_pat_abc123XYZ...`

### Optional: GitHub Container Registry (GHCR)
- **GHCR_TOKEN**: GitHub Personal Access Token for pushing to GHCR
  - Create at: https://github.com/settings/tokens
  - Required Scopes: `write:packages`, `read:packages`
  - Used in docker-build-push.yml (if multi-registry enabled)

## Variables (recommended for non-sensitive config)

Go to **Settings → Secrets and variables → Variables** to add:

| Variable | Value | Purpose |
|----------|-------|---------|
| `REGISTRY` | `docker.io` | Docker Hub registry URL |
| `IMAGE_BASE` | `username/discourse` | Base image name for tagging |
| `RUBY_VERSION` | `3.2` | Ruby version for test jobs |
| `NODE_VERSION` | `18` | Node.js version (if needed) |

## How to Add Secrets in GitHub UI

1. Go to your repository on GitHub
2. Click **Settings** (top right)
3. Click **Secrets and variables** → **Actions** (left sidebar)
4. Click **New repository secret**
5. Enter **Name**: `DOCKERHUB_USERNAME`
6. Enter **Secret**: Your Docker Hub username
7. Click **Add secret**
8. Repeat for `DOCKERHUB_TOKEN`

## How to Create a Docker Hub Access Token

1. Go to https://hub.docker.com/settings/security
2. Click **New Access Token**
3. Give it a memorable name (e.g., "GitHub Actions CI")
4. Select **Read & Write** permissions
5. Click **Generate**
6. Copy the token immediately (you won't see it again)
7. Paste it into GitHub as `DOCKERHUB_TOKEN`

## Testing Secrets Locally

To test your secrets work before committing:

```bash
# Log in to Docker with your token
echo "YOUR_DOCKERHUB_TOKEN" | docker login -u "YOUR_DOCKERHUB_USERNAME" --password-stdin

# Build and push test image
docker build -t YOUR_DOCKERHUB_USERNAME/discourse:test .
docker push YOUR_DOCKERHUB_USERNAME/discourse:test

# Log out
docker logout
```

## Security Best Practices

- ✅ Use **Personal Access Tokens**, not passwords
- ✅ Rotate tokens every 6-12 months
- ✅ Use **minimal permissions** (Read & Write only)
- ✅ Store secrets in GitHub Secrets, **never in .env or code**
- ✅ Review access logs at hub.docker.com/settings/security
- ✅ Delete unused tokens immediately
- ✅ Enable **branch protection** to require CI to pass before merging

## Troubleshooting Secrets

**"Provided credentials do not have permissions"**
- Verify token has "Read & Write" permissions
- Try regenerating the token

**"Unauthorized: authentication required"**
- Check DOCKERHUB_USERNAME and DOCKERHUB_TOKEN are spelled correctly
- Verify token hasn't expired (some token services auto-expire)

**"Error response from daemon"**
- Ensure Docker Hub account exists and credentials are valid
- Try logging in locally: `docker login -u USERNAME`

## Environment Variables vs Secrets

| Type | Use Case | Visibility |
|------|----------|-----------|
| **Secrets** | Credentials, tokens, passwords | Hidden in logs, never exposed |
| **Variables** | Non-sensitive config (Ruby version, etc.) | Visible in workflow runs |
| **.env files** | Local development only, **never commit** | Not used in GitHub Actions |

## Next Steps

1. Create secrets in GitHub (5 minutes)
2. Push to a test branch
3. Monitor **Actions** tab for workflow runs
4. Check build logs for any errors
5. Verify image appears on Docker Hub

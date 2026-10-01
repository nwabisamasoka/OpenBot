# Docker Image Registry & CI/CD Setup

## GitHub Actions CI/CD

A GitHub Actions workflow (`.github/workflows/docker-build.yml`) automatically builds and pushes images to Docker Hub on every push to `main` or `develop`.

### Setup Steps

1. **Create a Docker Hub account** (if you don't have one):
   - Go to https://hub.docker.com and sign up
   - Create a repository for each service: `openbot-agent-bot`, `openbot-agent-langgraph`, etc.

2. **Add GitHub Secrets**:
   - Go to your GitHub repo → **Settings** → **Secrets and variables** → **Actions**
   - Click **New repository secret** and add:
     - `DOCKER_USERNAME`: Your Docker Hub username
     - `DOCKER_PASSWORD`: Your Docker Hub access token (not your password)
       - Generate at https://hub.docker.com/settings/security

3. **Push to trigger the workflow**:
   ```bash
   git add .
   git commit -m "Add CI/CD pipeline"
   git push origin main
   ```
   - Go to your repo → **Actions** tab to monitor builds

### What the workflow does

- **Triggers on**: Push to `main`/`develop`, pull requests to `main`
- **Builds**: All 5 services (agent-bot, agent-langgraph, agent-computer, supervisor, migrate)
- **Tags images**: `latest` (main branch), git SHA, semantic versions (if you tag releases)
- **Pushes to**: `docker.io/YOUR_USERNAME/openbot-SERVICE:TAG`
- **Caches layers**: Reuses previous builds to speed up CI

### Using images in production

Pull and run from Docker Hub:

```bash
docker run docker.io/YOUR_USERNAME/openbot-agent-bot:latest
```

Update `docker-compose.yml` to pull from your registry:

```yaml
services:
  agent-bot:
    image: docker.io/YOUR_USERNAME/openbot-agent-bot:latest
    pull_policy: always
```

Then deploy with `docker compose up --pull always`.

## Manual Registry Push (if not using CI/CD)

```bash
docker tag openbot-agent-bot:latest docker.io/YOUR_USERNAME/openbot-agent-bot:latest
docker push docker.io/YOUR_USERNAME/openbot-agent-bot:latest
```

Repeat for each service.

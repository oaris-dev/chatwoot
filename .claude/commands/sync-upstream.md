# Sync Chatwoot Upstream

Sync the latest changes from the official Chatwoot repository into our fork while preserving customizations.

## Branch Strategy

```
upstream/develop → develop → chatwoot-oaris-edition
     ↑                ↑              ↑
  Official       Mirror with    Production with
  Chatwoot       CI/CD removed   customizations
```

## Instructions

Follow these steps to sync upstream changes:

### Step 1: Fetch and Review Upstream Changes

```bash
# Fetch latest from upstream
git fetch upstream develop

# Show new commits since last sync
git log --oneline upstream/develop --not origin/develop

# Count new commits
git rev-list --count origin/develop..upstream/develop
```

Review the changes and summarize:
- New features (feat:)
- Bug fixes (fix:)
- Breaking changes
- Database migrations (check db/migrate/)
- CI/CD changes (we skip these)

### Step 2: Merge to develop Branch

```bash
# Switch to develop
git checkout develop

# Merge upstream
git merge upstream/develop --no-edit
```

If there are conflicts in `.github/workflows/`:
- We removed upstream workflows (run_foss_spec.yml, publish_*.yml, etc.)
- Resolve by: `git rm <conflicting-workflow-file>`

Commit the merge with a descriptive message.

### Step 3: Push develop Branch

```bash
git push origin develop
```

### Step 4: Merge to Production Branch

```bash
# Switch to production branch
git checkout chatwoot-oaris-edition

# Merge develop
git merge develop -m "Merge develop into chatwoot-oaris-edition - vX.X.X update

Upstream changes:
- [list key features/fixes]

Customizations preserved:
- Logo size (h-12) on onboarding page
- Coolify labels removed
- Custom CI/CD workflow

Database migrations included:
- [list migration files if any]"
```

If there are conflicts in `.env.example` or `.gitignore`:
- Accept upstream version: `git checkout --theirs <file> && git add <file>`

### Step 5: Push and Deploy

```bash
# Push to trigger GitHub Actions build
git push origin chatwoot-oaris-edition
```

The push will:
1. Trigger GitHub Actions workflow (build-push-image.yml)
2. Build Docker image
3. Push to ghcr.io/oaris-dev/chatwoot:latest

### Step 6: Deploy via Coolify

1. **Staging (echo)**: Redeploy to test
2. **Production (alma)**: Redeploy after staging verification

Remember to use "Stop → Clean up old images → Redeploy" if Coolify uses cached images.

## Customizations to Preserve

These customizations are on `chatwoot-oaris-edition` and should NOT be overwritten:

1. **Logo size**: `app/views/installation/onboarding/index.html.erb` (h-8 → h-12)
2. **Coolify labels removed**: `docker-compose.yml` (no traefik labels)
3. **CI/CD**: `.github/workflows/build-push-image.yml` (our custom workflow)
4. **Removed workflows**: We deleted upstream CI workflows that don't work for our setup

## Workflows We Remove/Skip

These upstream workflows should be deleted if they cause conflicts:
- `.github/workflows/run_foss_spec.yml`
- `.github/workflows/run_mfa_spec.yml`
- `.github/workflows/publish_foss_docker.yml`
- `.github/workflows/publish_ee_docker.yml`
- `.github/workflows/frontend-fe.yml`
- `.github/workflows/test_docker_build.yml`

## Post-Sync Checklist

- [ ] New commits reviewed and summarized
- [ ] develop branch merged and pushed
- [ ] chatwoot-oaris-edition merged and pushed
- [ ] GitHub Actions build successful
- [ ] Staging (echo) deployed and tested
- [ ] Production (alma) deployed
- [ ] Version verified in Settings > Account Settings

## Current Version Info

After sync, verify the version in Chatwoot UI:
- Go to: Settings → Account Settings (bottom of page)
- Should show: `vX.X.X Build <commit-hash>`

The commit hash should match the latest commit on chatwoot-oaris-edition.

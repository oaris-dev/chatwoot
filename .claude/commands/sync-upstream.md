# Sync Chatwoot Upstream

Sync the latest changes from the official Chatwoot repository into our fork while preserving Oaris Edition customizations.

## Branch Strategy

```
upstream/develop → develop (mirror) → chatwoot-oaris-staging (test) → chatwoot-oaris-edition (production)
```

- `develop`: Clean mirror of upstream, only fork-specific removals (e.g. stale.yml)
- `chatwoot-oaris-staging`: Staging branch, deployed to echo for testing
- `chatwoot-oaris-edition`: Production branch, deployed to alma

## Important Notes

- The `develop` branch has a push protection hook (`bin/validate_push`). Use `--no-verify` when pushing — this is safe since we're intentionally syncing.
- Large upstream merges will trigger lint-staged / eslint on commit. Use `--no-verify` for merge commits — upstream code is already linted.
- **Never merge directly to `chatwoot-oaris-edition`**. Always go through staging first.
- After pushing staging, **wait for the user to verify echo** before merging to production.

## Instructions

### Step 1: Fetch and Review Upstream Changes

```bash
git fetch upstream develop

# Count and preview new commits
git rev-list --count origin/develop..upstream/develop
git log --oneline origin/develop..upstream/develop | head -30

# Check for version bumps
git log --oneline upstream/develop --grep="Bump version" | head -5

# Check for new migrations
git diff origin/develop..upstream/develop --name-only -- db/migrate/
```

Summarize for the user:
- How many commits behind
- Version releases included (e.g. v4.10.0 → v4.12.1)
- Notable features, fixes, and breaking changes
- New database migrations
- Any workflow/CI changes

### Step 2: Merge Upstream into develop

```bash
git checkout develop
git merge upstream/develop --no-edit
```

**After merging, remove unwanted workflows** that waste GitHub Actions runners on our fork:

```bash
# Remove if present — these are not needed on our fork:
git rm -f .github/workflows/stale.yml 2>/dev/null  # Auto-closes our PRs
# Add any other problematic workflows here as discovered
```

Commit the removal:
```bash
git commit --no-verify -m "chore: remove unwanted upstream workflows from fork"
```

If the merge itself had no workflow files to remove, skip this step.

### Step 3: Push develop

```bash
git push origin develop --no-verify
```

The `--no-verify` bypasses the `bin/validate_push` hook which blocks direct pushes to `develop` — this is safe for upstream syncs.

### Step 4: Merge develop into Staging

```bash
git checkout chatwoot-oaris-staging
git merge develop --no-edit
```

**Expected conflicts and how to resolve them:**

| File | Conflict reason | Resolution |
|------|----------------|------------|
| `config/app.yml` | We add `oaris_version` field | Keep upstream `version`, keep our `oaris_version` |
| `.github/workflows/stale.yml` | We deleted it, upstream modified | `git rm .github/workflows/stale.yml` |
| `.env.example` / `.gitignore` | Minor upstream changes | Accept upstream: `git checkout --theirs <file> && git add <file>` |

After resolving conflicts:
```bash
git add -A
git commit --no-verify -m "chore: sync with upstream vX.X.X

Upstream changes (N commits):
- [list key releases, features, fixes]

Customizations preserved:
- oaris_version in config/app.yml
- Custom build-push-image.yml workflow
- Removed stale.yml (not needed on fork)"
```

### Step 5: Push Staging and Wait for Verification

```bash
git push origin chatwoot-oaris-staging --no-verify
```

This triggers the GitHub Actions build for the `:staging` image.

**STOP HERE.** Tell the user:
- Staging is pushed and the image is building
- They should redeploy echo in Coolify (Stop → Clean up old images → Redeploy)
- Wait for them to confirm staging works before proceeding

**Do NOT proceed to Step 6 until the user confirms staging is working.**

### Step 6: Merge Staging into Production

Only after the user confirms staging works:

```bash
git checkout chatwoot-oaris-edition
git merge chatwoot-oaris-staging --no-edit
```

This should be a fast-forward merge if no other changes were made to production. If there are conflicts, resolve the same way as Step 4.

```bash
git push origin chatwoot-oaris-edition --no-verify
```

This triggers the GitHub Actions build for the `:latest` image.

**Check that only one build triggered** (not duplicates):
```bash
gh run list --branch chatwoot-oaris-edition --limit 3 -R oaris-dev/chatwoot
```

If both a push-triggered and manual-triggered run exist, cancel the manual one.

### Step 7: Deploy via Coolify

1. **Staging (echo)**: Already deployed and verified in Step 5
2. **Production (alma)**: Redeploy after `:latest` image build completes

Use "Stop → Clean up old images → Redeploy" if Coolify serves a cached old image.

## Customizations to Preserve

These files differ between `develop` and `chatwoot-oaris-edition` and must survive syncs:

| File | Customization |
|------|--------------|
| `config/app.yml` | `oaris_version: '1.0.0'` field added |
| `config/installation_config.yml` | Installation name set to "Chatwoot oaris edition" |
| `public/brand-assets/logo.svg` | Oaris Edition logo (light) |
| `public/brand-assets/logo_dark.svg` | Oaris Edition logo (dark) |
| `public/brand-assets/archive/` | Archived original Chatwoot logos |
| `app/views/installation/onboarding/index.html.erb` | Logo height h-8 → h-12 |
| `app/javascript/v3/views/auth/signup/Index.vue` | Login/signup branding |
| `app/javascript/v3/views/login/Index.vue` | Login branding |
| `app/javascript/v3/views/login/Saml.vue` | SAML login branding |
| `app/javascript/widget/i18n/locale/de.json` | German widget translations (du-form) |
| `app/controllers/api_controller.rb` | `oaris_version` in API response |
| `app/controllers/dashboard_controller.rb` | `OARIS_VERSION` exposed |
| `app/javascript/shared/store/globalConfig.js` | `oarisVersion` in store |
| `app/javascript/dashboard/.../BuildInfo.vue` | Oaris version display |
| `.github/workflows/build-push-image.yml` | Our custom GHCR image build |
| `.oaris/` | Project docs and deployment configs |
| `docker-compose.oaris.yaml` | Local dev / Coolify compose |

## Workflows We Remove

These should be deleted from our fork if upstream re-adds them:
- `.github/workflows/stale.yml` — Auto-closes our PRs after inactivity

The old list (run_foss_spec.yml, publish_foss_docker.yml, etc.) has been removed upstream as of v4.12.1 and is no longer a concern.

## Post-Sync Checklist

- [ ] Upstream changes reviewed and summarized
- [ ] `develop` merged and pushed
- [ ] Unwanted workflows removed
- [ ] `chatwoot-oaris-staging` merged, pushed, and image built
- [ ] Staging (echo) deployed and tested by user
- [ ] `chatwoot-oaris-edition` merged and pushed (only after staging verified)
- [ ] Production (alma) deployed
- [ ] Version verified in Settings → Account Settings
- [ ] Only one GitHub Actions build running (no duplicates)

# Oaris Chat (Chatwoot Fork) - Project Roadmap

**Repository:** `github.com/oaris-dev/chatwoot`
**Production Branch:** `chatwoot-oaris-edition`
**Upstream Sync Branch:** `develop`
**Docker Image:** `ghcr.io/oaris-dev/chatwoot:latest`

---

## 🎯 Milestone 1: Branding & Visual Identity

### Issue #1: Update Installation Name to "Chatwoot oaris edition" ✅ COMPLETE
- [x] Change `INSTALLATION_NAME` to "Chatwoot oaris edition" in `config/installation_config.yml:18`
- [x] Test if change is visible immediately or requires rebuild → **Requires rebuild + DB reset**
- [x] Verify browser tab title shows "Chatwoot oaris edition" ✅
- [x] Commit and push changes

**⚠️ FINDING: Configuration changes REQUIRE image rebuild**

**Why:** Configuration from `installation_config.yml` is:
1. Loaded only during `db:chatwoot_prepare` (first run)
2. Stored in PostgreSQL `installation_configs` table
3. Cached in Redis for 1 day
4. Cannot be hot-reloaded by container restart

**✅ Recommended Workflow for Config Changes:**

```bash
# 1. Edit config
vim config/installation_config.yml

# 2. Rebuild image (uses cache, ~5-10 min)
docker build -t oaris-chatwoot:latest -f docker/Dockerfile .

# 3. Reset database and start fresh (⚠️ LOSES ALL DATA)
docker-compose -f docker-compose.oaris.yaml down -v
docker-compose -f docker-compose.oaris.yaml up -d

# 4. Complete onboarding again
# Open http://localhost:3000 and verify branding
```

**⚠️ Important:** Config changes require fresh database (loses data). For production, config is set once during initial deployment.

**Production:** On Coolify, config is baked in during first deployment. To change, redeploy with new image.

---

### Issue #2: Document Logo Usage & Locations ✅ COMPLETE
- [x] Create comprehensive logo usage documentation → See `.oaris/local/LOGO-GUIDE.md`
- [x] List all logo file locations and their purposes
- [x] Document where each logo appears in the UI
- [x] Add recommended dimensions for each logo type

**Created:** `.oaris/local/LOGO-GUIDE.md` - Complete guide with specifications, testing checklist, and update workflow (local reference)

---

### Issue #3: Implement Oaris Edition Logos ✅ COMPLETE
- [x] Replace logo.svg and logo_dark.svg with Oaris Edition branded versions
- [x] New logos include "OA Edition" tagline (15.5K, viewBox 5570x1602)
- [x] Archive original Chatwoot logos for reference
- [x] Increase logo height from h-8 (32px) to h-12 (48px) in authentication pages
- [x] Test logos in login and signup pages
- [x] Commit and push changes

**Logo Inventory:**

| Logo File | Location | Used Where | Dimensions | Status |
|-----------|----------|------------|------------|--------|
| `logo_thumbnail.svg` | `/public/brand-assets/` | Favicon, widget footer, emails | 512×512px | ⚠️ Not customized yet |
| `logo.svg` | `/public/brand-assets/` | Dashboard, login (light mode) | Variable width | ✅ Oaris Edition |
| `logo_dark.svg` | `/public/brand-assets/` | Dashboard, login (dark mode) | Variable width | ✅ Oaris Edition |

**UI Locations:**
- ⚠️ **Favicon** - Browser tab icon (logo_thumbnail.svg) - Still default Chatwoot
- ✅ **Login page header** - Oaris Edition logo with tagline (48px height)
- ✅ **Signup page header** - Oaris Edition logo with tagline (48px height)
- ⚠️ **Dashboard header** - Not tested yet (likely still default)
- ⚠️ **Widget footer** - Not tested yet (likely still default)
- ⚠️ **Email templates** - Not tested yet (likely still default)

---

### Issue #4: Deployment Infrastructure ✅ COMPLETE
- [x] Create docker-compose.oaris.yaml for local development
- [x] Create .oaris/coolify-deployment.yaml for production
- [x] Add .gitignore rules for .oaris/local/ directory
- [x] Create comprehensive Coolify deployment guide
- [x] Commit and push deployment configurations

**Created Files:**
- `docker-compose.oaris.yaml` - Local development setup
- `.oaris/coolify-deployment.yaml` - Production deployment config
- `.oaris/local/COOLIFY-DEPLOYMENT-GUIDE.md` - Step-by-step deployment guide (11KB)

**Deployment Guide Includes:**
- Pre-deployment checklist (GitHub, domain, SMTP)
- All required environment variables with examples
- Commands to generate secure secrets
- Step-by-step Coolify setup instructions
- Troubleshooting common deployment issues
- Post-deployment monitoring and maintenance

---

## 🚀 Milestone 2: Production Deployment (Coolify)

### Issue #5: GitHub Container Registry Setup ✅ COMPLETE
- [x] Set up GitHub Actions workflow to build custom image
- [x] Configure automatic image build on push to `chatwoot-oaris-edition`
- [x] Publish to GitHub Container Registry (GHCR)
- [x] Update compose file to use GHCR image
- [x] Add Coolify labels to mark internal services

**Created:**
- `.github/workflows/build-push-image.yml` - Automated Docker image build
- Image: `ghcr.io/oaris-dev/chatwoot:latest`
- Build time: ~20-25 minutes (first build), ~10-15 minutes (cached)
- Cost: FREE (GitHub Actions + GHCR free for public repos)

**Benefits:**
- ✅ No build in Coolify (just pulls image)
- ✅ Faster deployments (~2-5 min vs ~20 min)
- ✅ Custom branding included in image
- ✅ Automatic rebuilds on code changes

---

### Issue #6: Set Up Coolify Service
- [ ] Make GHCR image public (required for Coolify to pull)
- [ ] Create new service in Coolify
- [ ] Configure as "Docker Compose" type
- [ ] Point to GitHub repo: `github.com/oaris-dev/chatwoot`
- [ ] Select branch: `chatwoot-oaris-edition`
- [ ] Specify docker-compose file: `.oaris/coolify-deployment.yaml`
- [ ] Set environment variables (SMTP, secrets, etc.)

**Coolify Configuration:**
- **Service Type:** Docker Compose
- **Repository:** `https://github.com/oaris-dev/chatwoot`
- **Branch:** `chatwoot-oaris-edition` (production with custom branding)
- **Docker Compose Location:** `.oaris/coolify-deployment.yaml`
- **Image:** `ghcr.io/oaris-dev/chatwoot:latest` (pre-built)

**Required Environment Variables:**
```env
# Secrets (generate with openssl rand -hex)
SERVICE_PASSWORD_CHATWOOT=<secret-key-base-128-chars>
SERVICE_PASSWORD_REDIS=<redis-password-64-chars>
SERVICE_USER_POSTGRES=chatwoot
SERVICE_PASSWORD_POSTGRES=<postgres-password-64-chars>
POSTGRES_DB=chatwoot

# URLs
SERVICE_URL_CHATWOOT=https://<your-domain>

# SMTP Configuration
CHATWOOT_MAILER_SENDER_EMAIL=<sender-email>
CHATWOOT_SMTP_ADDRESS=<smtp-server>
CHATWOOT_SMTP_PORT=587
CHATWOOT_SMTP_AUTHENTICATION=login
CHATWOOT_SMTP_USERNAME=<smtp-username>
CHATWOOT_SMTP_PASSWORD=<mailbox-password>
CHATWOOT_SMTP_DOMAIN=<your-domain>
CHATWOOT_SMTP_ENABLE_STARTTLS_AUTO=true
```

**First-Time Setup:**
1. Go to https://github.com/oaris-dev/chatwoot/pkgs/container/chatwoot
2. Make image public (Package settings → Change visibility → Public)

---

### Issue #7: Configure Domain & SSL ✅ MOSTLY COMPLETE
- [x] Set up domain (production domain) ✅
- [x] Configure SSL certificate (Let's Encrypt) ✅
- [x] Test HTTPS access ✅ HTTP/2 working
- [x] Verify branding ("Chatwoot oaris edition" with OA Edition logos) ✅
- [x] Create admin account and test login ✅ SuperAdmin account created
- [ ] Configure database backups → **POSTPONED to Issue #18**

**Status:** Production deployment live
**SSL:** Let's Encrypt certificate valid, auto-renewal configured
**Branding:** Verified working ("Chatwoot oaris edition" + OA Edition logos)
**Admin:** SuperAdmin account created and functional

---

## 🤖 Milestone 3: Flowise Integration

### Issue #8: Research Flowise Deployment Options
- [ ] Compare "Flowise with DB" vs "Flowise standalone" services
- [ ] Document differences and choose appropriate option
- [ ] Deploy Flowise instance on same Coolify server (vps alma)
- [ ] Document Flowise API endpoint URL
- [ ] Test Flowise API connectivity

**Flowise Service Options:**
1. **Flowise with DB** - Includes PostgreSQL, stores flow configurations
2. **Flowise Standalone** - External DB required, more flexible

**Network Setup:**
- **VPS:** alma (worker)
- **Manager:** nexus
- **Services:** Chatwoot by oaris + Flowise (same VPS)
- **Communication:** Internal Docker network or HTTP API

---

### Issue #9: Design Flowise Integration Architecture
- [ ] Define integration scope (inbox-level or account-level)
- [ ] Design message flow: Chatwoot → Flowise → Response
- [ ] Determine trigger conditions (all messages vs. specific inboxes)
- [ ] Plan error handling and fallback behavior
- [ ] Document API authentication method

**Proposed Architecture:**

```
┌─────────────┐
│  Customer   │
└──────┬──────┘
       │ Message
       ▼
┌─────────────────┐
│ Chatwoot Inbox  │
└──────┬──────────┘
       │ Trigger
       ▼
┌─────────────────────┐
│ Flowise Processor   │  (New Service)
│ Service             │
└──────┬──────────────┘
       │ HTTP Request
       ▼
┌─────────────────┐
│ Flowise API     │  (External Service)
│ (chat.oaris.ai) │
└──────┬──────────┘
       │ AI Response
       ▼
┌─────────────────┐
│ Chatwoot Reply  │
└─────────────────┘
```

**Integration Type:** `inbox` (one integration per inbox, not account-wide)

---

### Issue #10: Add Flowise to Integration Apps Config
- [ ] Add Flowise entry to `config/integration/apps.yml`
- [ ] Define settings schema (API key, endpoint URL)
- [ ] Create form schema for admin UI
- [ ] Add Flowise logo to `public/dashboard/images/integrations/flowise.png`

**File:** `config/integration/apps.yml`

**Add this block:**
```yaml
flowise:
  id: flowise
  logo: flowise.png
  i18n_key: flowise
  action: /flowise
  hook_type: inbox
  allow_multiple_hooks: false
  settings_json_schema:
    {
      'type': 'object',
      'properties':
        {
          'api_endpoint': { 'type': 'string' },
          'api_key': { 'type': 'string' },
          'flow_id': { 'type': 'string' },
        },
      'required': ['api_endpoint', 'flow_id'],
      'additionalProperties': false,
    }
  settings_form_schema:
    [
      {
        'label': 'Flowise API Endpoint',
        'type': 'text',
        'name': 'api_endpoint',
        'placeholder': 'https://flowise.oaris.ai',
        'validation': 'required',
      },
      {
        'label': 'API Key (optional)',
        'type': 'password',
        'name': 'api_key',
        'placeholder': 'Optional if no auth required',
      },
      {
        'label': 'Flow ID',
        'type': 'text',
        'name': 'flow_id',
        'placeholder': 'abc123-flow-id',
        'validation': 'required',
      },
    ]
  visible_properties: ['api_endpoint', 'flow_id']
```

---

### Issue #11: Create Flowise Processor Service
- [ ] Create `lib/integrations/flowise/processor_service.rb`
- [ ] Implement message event handling
- [ ] Add HTTP client for Flowise API calls
- [ ] Handle API responses and create reply messages
- [ ] Add error handling and logging

**File:** `lib/integrations/flowise/processor_service.rb`

**Key Methods:**
```ruby
class Integrations::Flowise::ProcessorService
  pattr_initialize [:event_name!, :hook!, :event_data!]

  def perform
    # 1. Get incoming message from event_data
    # 2. Check if message should trigger Flowise
    # 3. Call Flowise API with message content
    # 4. Parse response
    # 5. Create outgoing reply message in Chatwoot
  end

  private

  def call_flowise_api(message_text)
    # HTTP POST to Flowise endpoint
    # Include API key if configured
    # Return parsed response
  end

  def create_reply(conversation, response_text)
    # Create outgoing message in Chatwoot
  end
end
```

**Reference:** `lib/integrations/dialogflow/processor_service.rb` (similar pattern)

---

### Issue #12: Create Flowise Controller
- [ ] Create `app/controllers/api/v1/accounts/integrations/flowise_controller.rb`
- [ ] Implement setup action (create hook)
- [ ] Implement update action (modify settings)
- [ ] Implement destroy action (remove integration)
- [ ] Add to routes

**File:** `app/controllers/api/v1/accounts/integrations/flowise_controller.rb`

**Actions:**
- `POST /api/v1/accounts/:id/integrations/flowise` - Setup
- `PATCH /api/v1/accounts/:id/integrations/flowise/:hook_id` - Update
- `DELETE /api/v1/accounts/:id/integrations/flowise/:hook_id` - Remove

---

### Issue #13: Add Frontend Integration UI
- [ ] Add translations to `config/locales/en.yml`
- [ ] Add frontend translations to `app/javascript/dashboard/i18n/locale/en/integrations.json`
- [ ] Test integration setup flow in dashboard
- [ ] Verify settings form displays correctly

**Translations:**
```yaml
# config/locales/en.yml
integration_apps:
  flowise:
    name: 'Flowise'
    short_description: 'AI chatbot powered by Flowise'
    description: 'Connect Flowise AI flows to provide automated responses'
```

---

### Issue #14: Test Flowise Integration End-to-End
- [ ] Deploy Flowise instance
- [ ] Create test flow in Flowise
- [ ] Configure integration in Chatwoot inbox
- [ ] Send test messages
- [ ] Verify Flowise responses appear in conversation
- [ ] Test error scenarios (API down, invalid flow ID)
- [ ] Document any issues or limitations

---

## 📝 Additional Tasks

### Issue #15: Documentation
- [x] Create deployment guide for Coolify → `.oaris/local/COOLIFY-DEPLOYMENT-GUIDE.md`
- [x] Document logo replacement process → `.oaris/local/LOGO-GUIDE.md`
- [ ] Write Flowise integration setup guide
- [ ] Add upstream sync instructions
- [x] Document environment variables → In Coolify deployment guide

---

### Issue #16: Version Management
- [ ] Decide on version numbering scheme (e.g., 1.0.0-oaris)
- [ ] Create VERSION file or constant
- [ ] Display version in footer/about page
- [ ] Tag releases in GitHub

---

## 🔄 Maintenance Tasks

### Issue #17: Upstream Sync Strategy
- [ ] Set up periodic upstream sync schedule (monthly?)
- [ ] Document merge conflict resolution process
- [ ] Test sync with upstream Chatwoot
- [ ] Verify custom changes survive upstream merges

**Sync Commands:**
```bash
# Fetch upstream updates
git fetch upstream

# Merge into develop
git checkout develop
git merge upstream/develop

# Resolve conflicts in:
# - config/installation_config.yml (keep oaris changes)
# - config/integration/apps.yml (keep flowise entry)
# - .oaris/ directory (keep all)

# Rebuild and test
docker build -t oaris-chatwoot:latest -f docker/Dockerfile .
```

---

### Issue #18: Configure Database Backups
- [ ] Research Chatwoot built-in backup features
- [ ] Check if Chatwoot has native S3 backup integration
- [ ] Review Chatwoot backup documentation
- [ ] Compare Chatwoot native backups vs manual automation
- [ ] Decide on backup strategy (native vs Coolify Scheduled Tasks)
- [ ] Implement chosen backup solution
- [ ] Test backup and restore procedures
- [ ] Configure S3 lifecycle policies for retention

**Priority:** HIGH (should be completed within 24-48 hours of production deployment)

**Investigation needed:**
Before implementing manual backups via Coolify Scheduled Tasks, investigate if Chatwoot has:
- Built-in S3 backup integration
- Native backup commands or rake tasks
- Database migration/backup utilities
- Recommended backup procedures in docs

**Resources:**
- Backup automation plan: `.oaris/local/BACKUP-AUTOMATION-PLAN.md` (manual approach)
- Chatwoot docs: https://www.chatwoot.com/docs/self-hosted
- S3 already configured: Hetzner Object Storage (bucket: `oaris`)

**Options:**
1. **Chatwoot Native** - If available, use built-in backup features
2. **Coolify Scheduled Tasks** - Manual pg_dump + S3 upload (documented in backup plan)
3. **Hybrid** - Combine Chatwoot native + manual storage backups

---

## 📊 Progress Tracking

### Milestone 1: Branding & Visual Identity
- [x] Issue #1: Update installation name ✅
- [x] Issue #2: Document logo usage ✅
- [x] Issue #3: Implement Oaris Edition logos ✅
- [x] Issue #4: Deployment infrastructure ✅

### Milestone 2: Production Deployment
- [x] Issue #5: GitHub Container Registry setup ✅
- [x] Issue #6: Set up Coolify service ✅
- [x] Issue #7: Configure domain & SSL ✅ (backup task → Issue #18)

### Milestone 3: Flowise Integration
- [ ] Issue #8: Research Flowise deployment
- [ ] Issue #9: Design integration architecture
- [ ] Issue #10: Add to integration apps config
- [ ] Issue #11: Create processor service
- [ ] Issue #12: Create controller
- [ ] Issue #13: Add frontend UI
- [ ] Issue #14: End-to-end testing

### Additional
- [x] Issue #15: Documentation (Coolify guide + Logo guide) ✅
- [ ] Issue #16: Version management
- [ ] Issue #17: Upstream sync strategy
- [ ] Issue #18: Configure database backups (HIGH priority)

---

## 🚦 Current Status

**Last Updated:** 2025-11-12

**Current Phase:** 🎉 **Milestone 2 COMPLETE** → Production Deployment Live!
**Production Branch:** `chatwoot-oaris-edition`
**Upstream Sync Branch:** `develop` (mirrors upstream Chatwoot)
**Docker Image:** `ghcr.io/oaris-dev/chatwoot:latest` (GitHub Container Registry)

**Branching Strategy:**
```
upstream/develop → develop (clean mirror) → chatwoot-oaris-edition (customizations)
```

**✅ Milestone 1 COMPLETE:**
- [x] Fork Chatwoot repository
- [x] Set up Git workflow (upstream + origin with proper branch strategy)
- [x] Change installation name to "Chatwoot oaris edition"
- [x] Implement Oaris Edition logos with "OA Edition" tagline
- [x] Increase logo visibility (32px → 48px)
- [x] Create deployment configurations (Coolify)
- [x] Document logo usage and deployment process

**✅ Milestone 2 COMPLETE:**
- [x] Set up GitHub Actions for automated image builds
- [x] Configure GitHub Container Registry publishing
- [x] Add Coolify service labels for internal services
- [x] Deploy to production
- [x] Configure SSL certificate (Let's Encrypt)
- [x] Verify custom branding in production
- [x] Create SuperAdmin account

**🚀 Production Deployment:**
- ✅ **SSL:** Let's Encrypt certificate with HTTP/2
- ✅ **Branding:** "Chatwoot oaris edition" verified
- ✅ **Admin:** SuperAdmin account functional
- ✅ **Infrastructure:** 4 healthy containers (chatwoot, sidekiq, postgres, redis)
- ✅ **Resources:** 773 MB RAM total (very efficient)
- ✅ **Image:** Custom GHCR image with auto-builds

**🔜 Next Up (Milestone 3):**
1. **Issue #18:** Research and configure database backups (HIGH priority)
2. **Issue #8-14:** Flowise Integration (AI chatbot powered by Flowise flows)

---

## 📞 Questions & Answers

### Q: Can I change configuration without rebuilding?
**A:** Test needed! Configuration from `installation_config.yml` is loaded into database on first run. Changes might require:
1. Container restart (try this first)
2. Database migration/seed
3. Full rebuild (worst case)

### Q: Can Coolify use custom docker-compose file from repo?
**A:** ✅ Yes! Coolify supports specifying both Git repo and custom docker-compose path.

### Q: Flowise with DB vs standalone?
**A:** Need to research. Likely:
- **With DB**: Easier setup, stores flows in PostgreSQL
- **Standalone**: More flexible, can use external DB

---

## 🔗 Useful Links

- **Repository:** https://github.com/oaris-dev/chatwoot
- **Upstream:** https://github.com/chatwoot/chatwoot
- **Chatwoot Docs:** https://www.chatwoot.com/docs
- **Flowise Docs:** https://docs.flowiseai.com
- **Coolify Docs:** https://coolify.io/docs

---

**Document Version:** 1.0
**Maintained by:** oaris development team

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

### Issue #19: Investigate & Implement Email Template Branding
- [ ] Identify all mailer classes (`app/mailers/`)
- [ ] Identify all email view templates (`app/views/*_mailer/`, `app/views/mailers/`)
- [ ] Check email layout files for shared header/footer with logo
- [ ] Check if `logo_thumbnail.svg` is used in emails (currently not customized)
- [ ] Check if `INSTALLATION_NAME` is used in email copy or hardcoded as "Chatwoot"
- [ ] Review Enterprise email overrides (`enterprise/app/views/`)
- [ ] Check CSAT survey email templates
- [ ] Replace email logo with Oaris Edition branding
- [ ] Ensure installation name is used dynamically (not hardcoded)
- [ ] Update `logo_thumbnail.svg` if used in emails
- [ ] Test all email types render correctly with new branding

**Related:**
- Issue #3 notes `logo_thumbnail.svg` is not customized yet
- Issue #2 / `.oaris/local/LOGO-GUIDE.md` documents logo locations including emails
- `app/mailers/` — mailer classes
- `app/views/` — email templates
- `config/installation_config.yml` — installation name config

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
       │ Message (text or postback from button click)
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
│ (Mistral LLM)   │
└──────┬──────────┘
       │ AI Response (text, cards, forms, or handoff)
       ▼
┌──────────────────────┐
│ Chatwoot Reply       │
│ (text / cards / form)│
└──────────────────────┘
```

**Integration Type:** `inbox` (one integration per inbox, not account-wide)
**LLM:** Mistral AI (via Flowise, self-hosted)

**Rich Message Support:**

Flowise can return structured responses that the processor service maps to Chatwoot content types:

| Flowise response type | Chatwoot content_type | Use case |
|---|---|---|
| Plain text | `text` | Simple answers, FAQs |
| Product/service card | `cards` | Hosting packages, pricing, proposals with "Buy" button |
| Data collection | `form` | Billing details, contact info, order forms |
| Options list | `input_select` | Choose a plan, pick a time slot |
| Handoff signal | (transfer to agent) | Complex queries, complaints |

**Example — Hosting package card:**
```json
{
  "content_type": "cards",
  "content_attributes": {
    "items": [
      {
        "media_url": "https://oaris.de/images/hosting-pro.png",
        "title": "Professional Hosting",
        "description": "50 GB SSD, 5 Domains, SSL included — €9.90/month",
        "actions": [
          { "type": "link", "text": "Details", "uri": "https://oaris.de/hosting/pro" },
          { "type": "postback", "text": "Buy Now", "payload": "BUY_HOSTING_PRO" }
        ]
      },
      {
        "media_url": "https://oaris.de/images/hosting-business.png",
        "title": "Business Hosting",
        "description": "200 GB SSD, Unlimited Domains, Priority Support — €24.90/month",
        "actions": [
          { "type": "link", "text": "Details", "uri": "https://oaris.de/hosting/business" },
          { "type": "postback", "text": "Buy Now", "payload": "BUY_HOSTING_BUSINESS" }
        ]
      }
    ]
  }
}
```

**Postback flow:** When customer clicks "Buy Now", the postback payload (`BUY_HOSTING_PRO`) is sent back as a new message → triggers Flowise again → Flowise can respond with a form to collect billing details or a payment link.

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
- [ ] Implement message event handling (text + postback payloads)
- [ ] Add HTTP client for Flowise API calls
- [ ] Handle plain text responses → `content_type: text`
- [ ] Handle structured responses → `content_type: cards`, `form`, `input_select`
- [ ] Handle handoff signals → transfer conversation to human agent
- [ ] Add error handling and logging

**File:** `lib/integrations/flowise/processor_service.rb`

**Key Methods:**
```ruby
class Integrations::Flowise::ProcessorService
  pattr_initialize [:event_name!, :hook!, :event_data!]

  def perform
    # 1. Get incoming message from event_data
    # 2. Check if message should trigger Flowise (skip outgoing, activity, etc.)
    # 3. Extract message content (text body or postback payload)
    # 4. Call Flowise API with message content
    # 5. Parse response type (text, cards, form, handoff)
    # 6. Create appropriate Chatwoot reply message
  end

  private

  def call_flowise_api(message_text)
    # HTTP POST to #{hook.settings['api_endpoint']}/api/v1/prediction/#{hook.settings['flow_id']}
    # Include API key header if hook.settings['api_key'] present
    # Return parsed response
  end

  def create_text_reply(conversation, text)
    # content_type: 'text', content: text
  end

  def create_cards_reply(conversation, items)
    # content_type: 'cards', content_attributes: { items: [...] }
    # Each item: { media_url, title, description, actions: [{type, text, uri/payload}] }
  end

  def create_form_reply(conversation, fields)
    # content_type: 'form', content_attributes: { items: [...] }
  end

  def handoff_to_agent(conversation)
    # Set conversation status to 'open' (removes bot, alerts agents)
  end
end
```

**Response mapping:** Flowise returns JSON. The processor service inspects the response
to determine message type. Convention TBD — options:
1. Flowise returns a structured JSON with `type` field (`text`, `cards`, `form`, `handoff`)
2. Flowise returns plain text by default, structured content via special JSON format

**Reference:** `lib/integrations/dialogflow/processor_service.rb` (similar pattern, plus
the new `language_code` configurable approach from PR #13221)

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

### Issue #16: Version Management ✅ COMPLETE
- [x] Decide on version numbering scheme → `oaris_version: '1.0.0'`
- [x] Create VERSION constant → `config/app.yml`
- [x] Display version in footer/about page → `BuildInfo.vue` shows "Oaris v1.0.0"
- [x] Expose version in API → `/api` returns `oaris_version`
- [ ] Tag releases in GitHub (future)

**Version Scheme:**
- Upstream Chatwoot version: `4.8.0` (from `version` in `config/app.yml`)
- Oaris Edition version: `1.0.0` (from `oaris_version` in `config/app.yml`)

**Display Format:**
- Settings → Account Settings: `v4.8.0 | Oaris v1.0.0 | Build abc1234`
- API endpoint `/api`: `{"version": "4.8.0", "oaris_version": "1.0.0", ...}`

**Files Modified:**
- `config/app.yml` - Added `oaris_version: '1.0.0'`
- `app/controllers/dashboard_controller.rb` - Exposed `OARIS_VERSION`
- `app/controllers/api_controller.rb` - Added `oaris_version` to API response
- `app/javascript/shared/store/globalConfig.js` - Added `oarisVersion` to state
- `app/javascript/dashboard/routes/dashboard/settings/account/components/BuildInfo.vue` - Display Oaris version

---

## 🔄 Maintenance Tasks

### Issue #17: Upstream Sync Strategy ✅ COMPLETE
- [x] Set up periodic upstream sync schedule (monthly?)
- [x] Document merge conflict resolution process
- [x] Test sync with upstream Chatwoot
- [x] Verify custom changes survive upstream merges
- [x] Create Claude Code slash command for easy syncing

**Slash Command:** `/sync-upstream`
- Location: `.claude/commands/sync-upstream.md`
- Provides step-by-step guide for merging upstream changes
- Handles workflow conflicts automatically
- Preserves customizations (logos, CI/CD)

**Branch Strategy:**
```
upstream/develop → develop (mirror, CI/CD removed) → chatwoot-oaris-edition (production)
```

**Syncs Completed:**
- 2025-11-28: v4.8.0 + 20 additional commits (Pinia, voice calls, Captain instrumentation)

---

### Issue #18: Configure Database Backups ✅ COMPLETE
- [x] Research Chatwoot built-in backup features
- [x] Check if Chatwoot has native S3 backup integration
- [x] Review Chatwoot backup documentation
- [x] Compare Chatwoot native backups vs manual automation
- [x] Decide on backup strategy (system crontab with shell scripts)
- [x] Implement backup scripts on alma server
- [x] Test backup and restore procedures
- [ ] Configure S3 lifecycle policies for retention (optional - manual cleanup works)

**Implementation:** System crontab with shell scripts on alma server

**Scripts Created:**
- `/opt/chatwoot-backup-postgres.sh` - PostgreSQL backup (dynamically detects user)
- `/opt/chatwoot-backup-storage.sh` - Rails storage volume backup

**Schedule (crontab on alma):**
- `0 2 * * *` - PostgreSQL backup at 2 AM UTC daily
- `0 3 * * *` - Rails storage backup at 3 AM UTC daily

**S3 Storage:**
- PostgreSQL: `s3://oaris/chatwoot/production/postgres/`
- Rails storage: `s3://oaris/chatwoot/production/storage/`

**Logs:** `/var/log/chatwoot-backup.log`

**Commands:**
```bash
# Manual backup
ssh alma 'sudo /opt/chatwoot-backup-postgres.sh'
ssh alma 'sudo /opt/chatwoot-backup-storage.sh'

# Check logs
ssh alma 'cat /var/log/chatwoot-backup.log'
```

**Full Documentation:** `.oaris/local/BACKUP-AUTOMATION-PLAN.md` (restore procedures, S3 commands, verification steps)

**Why System Crontab Instead of Coolify:**
- Coolify Scheduled Tasks had a 255-character command limit bug
- System crontab is more reliable and persists across Coolify updates
- Scripts are easier to maintain and update

---

## 📊 Progress Tracking

### Milestone 1: Branding & Visual Identity
- [x] Issue #1: Update installation name ✅
- [x] Issue #2: Document logo usage ✅
- [x] Issue #3: Implement Oaris Edition logos ✅
- [ ] Issue #19: Investigate & implement email template branding
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
- [x] Issue #16: Version management ✅
- [x] Issue #17: Upstream sync strategy ✅
- [x] Issue #18: Configure database backups ✅

---

## 🚦 Current Status

**Last Updated:** 2025-11-30

**Current Phase:** 🎉 **Milestone 2 COMPLETE** → Production Deployment Live!
**Chatwoot Version:** v4.8.0 (synced with upstream 2025-11-28)
**Oaris Edition:** v1.0.0
**Production Branch:** `chatwoot-oaris-edition`
**Staging Branch:** `chatwoot-oaris-staging`
**Upstream Sync Branch:** `develop` (mirrors upstream Chatwoot)
**Docker Image:** `ghcr.io/oaris-dev/chatwoot:latest` (GitHub Container Registry)

**Branching Strategy:**
```
upstream/develop → develop (clean mirror) → chatwoot-oaris-staging (testing) → chatwoot-oaris-edition (production)
                                                    ↑
                                            feature/* branches
```

**Workflow:**
1. Create feature branches from `chatwoot-oaris-staging`
2. Test features on staging deployment
3. Merge to `chatwoot-oaris-staging` for integration testing
4. Promote to `chatwoot-oaris-edition` for production

**✅ Milestone 1 COMPLETE:**
- [x] Fork Chatwoot repository
- [x] Set up Git workflow (upstream + origin with proper branch strategy)
- [x] Change installation name to "Chatwoot oaris edition"
- [x] Implement Oaris Edition logos with "OA Edition" tagline
- [x] Increase logo visibility (32px → 48px)
- [x] Create deployment configurations (Coolify)
- [x] Document logo usage and deployment process
- [x] Complete German widget translations (du-form)

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

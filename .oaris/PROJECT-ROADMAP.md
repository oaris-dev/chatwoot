# Oaris Chat (Chatwoot Fork) - Project Roadmap

**Repository:** `github.com/oaris-dev/chatwoot`
**Branch:** `feature/oaris-branding`
**Local Instance:** http://localhost:3000

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

### Issue #5: Prepare for Coolify Deployment
- [ ] Merge `feature/oaris-branding` into `develop` branch
- [ ] Push `develop` to GitHub
- [ ] Verify all deployment configs are committed
- [ ] Test GitHub repo accessibility from Coolify server

**Pre-deployment Checklist:**
```bash
# Merge feature branch
git checkout develop
git merge feature/oaris-branding
git push origin develop

# Verify deployment files exist
ls -la .oaris/coolify-deployment.yaml
ls -la docker-compose.oaris.yaml
```

---

### Issue #6: Set Up Coolify Service
- [ ] Create new service in Coolify
- [ ] Configure as "Docker Compose" type
- [ ] Point to GitHub repo: `github.com/oaris-dev/chatwoot`
- [ ] Select branch: `develop`
- [ ] Specify docker-compose file: `.oaris/coolify-deployment.yaml`

**Coolify Configuration:**
- **Service Type:** Docker Compose
- **Repository:** `https://github.com/oaris-dev/chatwoot`
- **Branch:** `develop`
- **Docker Compose Location:** `.oaris/coolify-deployment.yaml`
- **Build Context:** `.` (root)
- **Dockerfile:** `docker/Dockerfile`

**Environment Variables to Set:**
```env
SERVICE_PASSWORD_CHATWOOT=<generate-secret-key>
SERVICE_URL_CHATWOOT=https://chat.oaris.dev
SERVICE_PASSWORD_REDIS=<generate-redis-password>
SERVICE_USER_POSTGRES=chatwoot
SERVICE_PASSWORD_POSTGRES=<generate-postgres-password>
POSTGRES_DB=chatwoot
CHATWOOT_MAILER_SENDER_EMAIL=noreply@oaris.dev
```

**Question to Resolve:** ✅ Yes, Coolify supports specifying both Git repo AND custom docker-compose file path

---

### Issue #7: Configure Domain & SSL
- [ ] Set up domain: `chat.oaris.dev` (or chosen domain)
- [ ] Configure SSL certificate (Let's Encrypt)
- [ ] Test HTTPS access
- [ ] Verify redirects and CORS settings

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

## 📊 Progress Tracking

### Milestone 1: Branding & Visual Identity
- [x] Issue #1: Update installation name ✅
- [x] Issue #2: Document logo usage ✅
- [x] Issue #3: Implement Oaris Edition logos ✅
- [x] Issue #4: Deployment infrastructure ✅

### Milestone 2: Production Deployment
- [ ] Issue #5: Prepare for deployment
- [ ] Issue #6: Set up Coolify service
- [ ] Issue #7: Configure domain & SSL

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

---

## 🚦 Current Status

**Last Updated:** 2025-11-11

**Current Phase:** Milestone 1 COMPLETE ✅ → Ready for Milestone 2 (Production Deployment)
**Active Branch:** `feature/oaris-branding`
**Local Instance:** Running at http://localhost:3000
**Docker Image:** `oaris-chatwoot:latest` (built with logos and branding)

**Milestone 1 Completed:**
- [x] Fork Chatwoot repository
- [x] Set up Git workflow (upstream + origin)
- [x] Change installation name to "Chatwoot oaris edition"
- [x] Implement Oaris Edition logos with "OA Edition" tagline
- [x] Increase logo visibility (32px → 48px)
- [x] Build custom Docker image
- [x] Local development environment running
- [x] Create deployment configurations (local + production)
- [x] Document logo usage and deployment process
- [x] Clean git history (4 logical commits)

**What's Working:**
- ✅ Browser tab: "Chatwoot oaris edition"
- ✅ Login page: Oaris Edition logo (48px height)
- ✅ Signup page: Oaris Edition logo (48px height)
- ✅ Local development: http://localhost:3000
- ✅ Docker compose files ready for deployment

**Next Up (Milestone 2):**
1. **Issue #5:** Merge `feature/oaris-branding` into `develop`
2. **Issue #6:** Set up Coolify service (follow `.oaris/local/COOLIFY-DEPLOYMENT-GUIDE.md`)
3. **Issue #7:** Configure domain & SSL (chat.oaris.dev)

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

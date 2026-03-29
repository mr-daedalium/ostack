# /startup Skill Design Spec

**Date:** 2026-03-29
**Status:** Draft
**Author:** User + Claude (brainstorming session)

---

## Overview

`/startup` is an ostack skill that takes a user from an empty folder to a fully provisioned, deployed, and documented project. It handles project identity, architecture advisory, GCP infrastructure, third-party service setup, code scaffolding, and file generation — all in a single invocation.

It is **not** a brainstorming tool (that's `/office-hours`). It is operational: collect facts, make architecture decisions, provision everything, generate code, push to GitHub.

---

## Approach

**Approach B: Router Skill + Route Modules** (selected over monolithic and two-skill alternatives).

A main `SKILL.md.tmpl` orchestrates the flow. Route-specific architecture topics, provisioning steps, and generators live in separate module files read by the main skill at the appropriate phase.

---

## Skill File Structure

```
startup/
├── SKILL.md.tmpl                  # Main skill: interview + router + orchestrator
├── routes/
│   ├── saas.md                    # SaaS web app architecture topics + scaffold spec
│   ├── landing.md                 # Landing page / marketing site
│   ├── ai.md                      # AI model training / ML project
│   ├── internal-tool.md           # Internal tools
│   ├── api.md                     # API service / backend
│   └── cli.md                     # CLI tool route
├── provisioning/
│   ├── gcp-project.md             # GCP project, billing, service account, APIs
│   ├── github.md                  # GitHub repo creation
│   ├── dns.md                     # Cloud DNS zone + nameserver output
│   ├── cloud-run.md               # Cloud Run service + custom domain + SSL
│   ├── analytics.md               # GA4 property + Search Console verification
│   ├── posthog.md                 # PostHog project creation via MCP
│   ├── stripe.md                  # Stripe full setup via MCP (test mode)
│   ├── email.md                   # Resend key + domain DNS + welcome template
│   ├── monitoring.md              # GCP Uptime Check + Cloud Monitoring alerts + Sentry
│   └── cicd.md                    # CI/CD pipeline (GitHub Actions or Cloud Build)
├── generators/
│   ├── readme.md                  # README generation with ASCII art + best practices
│   ├── project-json.md            # project.json schema + PROJECT.md generation
│   ├── claude-md.md               # CLAUDE.md seeding rules
│   ├── adr.md                     # Architecture Decision Record format
│   ├── gitignore.md               # Stack-tailored .gitignore rules
│   └── scaffold.md                # Working skeleton generation per route
└── discovery/
    └── service-suggestions.md     # "Did you forget about..." smart scan
```

Each sub-file is a **prompt fragment** — instructions for Claude, not standalone skills. The main `SKILL.md.tmpl` reads them via the `Read` tool at the right moment.

---

## User Defaults (Hardcoded)

These are the user's permanent preferences. The skill does not ask about them.

| Setting | Value |
|---------|-------|
| Hosting | Google Cloud Run (default; flag if not adapted) |
| SSL | Cloud Run managed certificates |
| DNS management | Google Cloud DNS (CLI-only, no web console) |
| Domain registrar | Namecheap (manual purchase, skill creates DNS zone) |
| GCP billing | Single billing account |
| GCP service account role | Owner |
| GCP project ID convention | Same as project slug |
| Cloud Run service name | `{slug}-{env}` (e.g., `my-app-prod`) |
| Service account key storage | `~/.gcloud-keys/{project-id}.json` (chmod 600) |
| Secrets | `.env` locally (gitignored) + GCP Secret Manager for production |
| PostHog | Cloud, separate org per project |
| Stripe | Always test mode first |
| Error tracking | PostHog error tracking (not standalone Sentry) |
| Uptime monitoring | GCP Uptime Checks |
| Log alerts | GCP Cloud Monitoring alert policies |
| Linter/formatter | Stack-appropriate (Biome for JS/TS, ruff for Python, rustfmt for Rust) |
| `/startup` scope | Day-zero only (not for retrofitting existing projects) |
| Provisioning mode | Full auto after architecture sign-off (no per-step confirmations) |
| Scaffold depth | Working skeleton (builds, runs, deploys, health check — no business logic) |

---

## The Flow

### Phase 1: Interview (Steps 1-2)

#### Step 1: Project Basics

Asked via AskUserQuestion, **one at a time**:

1. **Project name** — human-readable (e.g., "My SaaS App")
2. **Project slug** — auto-suggested from name. Used for GCP project ID, Cloud Run service, folder name. User can override.
3. **One-line description** — what the project does
4. **Project type** — pick one:
   - SaaS web app
   - Landing page / marketing site
   - AI / model training
   - Internal tool
   - API service
   - CLI tool
5. **Paid or free?** — if paid, Stripe setup activates
6. **Needs email sending?** — if yes, Resend setup activates
7. **Target audience geography** — informs GCP region recommendation
8. **GCP region** — recommended based on geography, user confirms
9. **Domain name** — already bought or "none yet"
10. **GitHub** — public or private? Which organization?
11. **Environments** — just production? Staging? Dev?

#### Step 2: Architecture Conversation

The skill loads the route-specific file (e.g., `routes/saas.md`) and walks through each architecture topic **one at a time**.

For each topic:
- Make a recommendation with reasoning
- Present 1-2 alternatives with trade-offs
- Wait for user sign-off before moving on

**Common topics (all routes):**

1. Language & framework
2. Database (type, hosting, ORM explanation, recommendation)
3. Authentication approach
4. Hosting & scaling (Cloud Run default, flag if not adapted for this project type)
5. CI/CD approach (GitHub Actions vs Cloud Build, discuss trade-offs)

**Route-specific topics:**

| Route | Extra architecture topics |
|-------|--------------------------|
| **SaaS** | Multi-tenancy model, subscription/billing logic (free tier? trial? seat-based?), user roles & permissions, onboarding flow, GDPR/data export |
| **Landing page** | Static vs SSR (does it even need Next.js?), CMS choice (hardcoded, Notion, Contentful, MDX), form handling (where data goes), SEO strategy, performance budget (Core Web Vitals) |
| **AI / ML** | GPU strategy (Cloud GPUs, Lambda Labs, RunPod, local), dataset storage & versioning (GCS, DVC, HuggingFace), experiment tracking (W&B, MLflow), model serving & inference, inference cost estimation, open-source licensing implications |
| **Internal tool** | Access control (IAP, Google Workspace SSO, VPN), build vs buy (Retool?), data sources (existing DBs, APIs, spreadsheets), does it need to be pretty or just functional? |
| **API service** | API spec format (OpenAPI/Swagger from day one), rate limiting, API key management for consumers, versioning strategy (URL path, headers), SDK generation for consumers |
| **CLI tool** | Distribution (npm, Homebrew, binary releases), language choice (Rust for speed, Node for ecosystem, Go for single binary), config file format & location, update mechanism (self-update, package manager), shell completions |

Each decision is queued as an ADR entry (written later in Step 14).

#### Service Discovery Scan

After architecture conversation, before provisioning:

> "Based on what we discussed, you might also need: [smart list]. Want to add any?"

Detection logic examples:
- User uploads mentioned → Cloud Storage + CDN
- Multi-tenant SaaS → feature flags (PostHog has this)
- AI inference → async job queue (Cloud Tasks)
- International users → Cloud CDN + i18n
- File processing → Cloud Functions
- Real-time features → Cloud Pub/Sub or WebSockets
- Search → Algolia or Elasticsearch
- Caching/rate limiting → Memorystore (Redis)
- Data analytics pipeline → BigQuery
- Location-based features → Maps/Places API

User picks what to add. Anything added gets its own provisioning step and appropriate GCP APIs enabled.

---

### Phase 2: Provisioning (Steps 3-13)

**Runs fully automated** after architecture sign-off. No per-step confirmations.

**Idempotency rule:** Every step checks if the resource exists first. If it does, skip and log "already exists." This makes `/startup` safe to re-run on partially provisioned projects.

**Error handling:** Any step that fails logs the error, continues to the next step, and reports all failures at the end. User can re-run `/startup` to retry failed steps.

#### Step 3: GitHub

```
- Check if git is initialized, if not: git init
- Check if repo exists: gh repo view {org}/{slug} 2>/dev/null
- If not: gh repo create {slug} --{public|private} --org {org} --source . --push
- If exists: skip, log "repo already exists"
```

#### Step 4: GCP Project

```
- Check gcloud auth: gcloud auth list --filter=status:ACTIVE --format="value(account)"
- If not logged in: prompt user to run "! gcloud auth login"
- Check if project exists: gcloud projects describe {slug} 2>/dev/null
- If not: gcloud projects create {slug} --name="{project name}"
- Detect billing account: gcloud billing accounts list --format="value(name)" --limit=1
- Link billing: gcloud billing projects link {slug} --billing-account={detected-billing-account}
- Set active project: gcloud config set project {slug}

Always-enable APIs (single batch):
  - run.googleapis.com
  - cloudbuild.googleapis.com
  - storage.googleapis.com
  - dns.googleapis.com
  - artifactregistry.googleapis.com
  - secretmanager.googleapis.com
  - logging.googleapis.com
  - iam.googleapis.com
  - cloudscheduler.googleapis.com
  - cloudtasks.googleapis.com

Conditional APIs (based on architecture decisions):
  - sqladmin.googleapis.com (if Cloud SQL)
  - firestore.googleapis.com (if Firestore)
  - compute.googleapis.com (if AI/GPU workloads)
  - aiplatform.googleapis.com (if Vertex AI)
  - cloudfunctions.googleapis.com (if Cloud Functions needed)
  - pubsub.googleapis.com (if event-driven)
  - redis.googleapis.com (if caching/rate limiting)
  - bigquery.googleapis.com (if data pipeline)
  - translate.googleapis.com / vision.googleapis.com (if Google AI APIs)

Service account:
  - gcloud iam service-accounts create {slug}-sa --display-name="{project name} Service Account"
  - gcloud projects add-iam-policy-binding {slug} --member="serviceAccount:{slug}-sa@{slug}.iam.gserviceaccount.com" --role="roles/owner"
  - gcloud iam service-accounts keys create ~/.gcloud-keys/{slug}.json --iam-account={slug}-sa@{slug}.iam.gserviceaccount.com
  - chmod 600 ~/.gcloud-keys/{slug}.json
```

#### Step 5: Cloud DNS

```
- Check if zone exists: gcloud dns managed-zones describe {slug}-zone 2>/dev/null
- If not: gcloud dns managed-zones create {slug}-zone --dns-name="{domain}." --description="{project name}"
- Extract nameservers
- Display: "Set these nameservers in Namecheap for {domain}: ns1... ns2... ns3... ns4..."
- This is the ONE expected manual step
```

#### Step 6: Cloud Run + Domain + SSL

```
- Deploy minimal health-check container to Cloud Run in chosen region
- Map custom domain: gcloud run domain-mappings create --service {slug} --domain {domain} --region {region}
- SSL provisioned automatically by Cloud Run
- Verify service responds on Cloud Run URL
```

#### Step 7: Scaffold Code

```
- Load generators/scaffold.md + route-specific scaffold spec
- Generate working skeleton based on route + all architecture decisions:
  - Dockerfile + .dockerignore
  - Health endpoint (GET /health or GET /api/health)
  - Basic project structure per framework
  - Package manager config (package.json, pyproject.toml, Cargo.toml, etc.)
  - Linter/formatter config (biome.json, ruff.toml, rustfmt.toml, etc.)
- The skeleton MUST: build, run locally, deploy to Cloud Run, pass a health check
- No business logic — just a working shell
```

#### Step 8: GA4

```
- Use Google Analytics Admin API:
  - Create GA4 property for the domain
  - Create web data stream
  - Extract measurement ID
- Install tracking code in scaffold (framework-appropriate: next/script, meta tag, etc.)
- Store measurement ID in project.json + .env
- If API fails: log error, provide manual fallback instructions
```

#### Step 9: Google Search Console

```
- Use Search Console API
- Add DNS TXT verification record to Cloud DNS zone
- Verify site ownership
- Store verification status in project.json
- If API fails: log error, provide manual fallback instructions
```

#### Step 10: PostHog

```
- Use PostHog MCP tools:
  - Create new organization for this project
  - Create project within it
  - Extract project API key and host
- Install PostHog SDK in scaffold (framework-appropriate)
- Enable error tracking in the project
- Store org ID, project ID, API key in project.json + .env
```

#### Step 11: Stripe (conditional — if paid)

```
- Use Stripe MCP tools (test mode):
  - Verify account info
  - Create product with project name + description
  - Create price (based on billing model from architecture phase)
  - Create webhook endpoint → {domain}/api/webhooks/stripe
  - Configure customer portal
- Scaffold in code:
  - Webhook handler endpoint
  - Billing/pricing page
  - Customer portal redirect
- Store keys in .env:
  - STRIPE_SECRET_KEY (test)
  - NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY (test)
  - STRIPE_WEBHOOK_SECRET
- Store product/price IDs in project.json
```

#### Step 12: Resend (conditional — if email)

```
- Store Resend API key in .env
- Add DNS records to Cloud DNS zone:
  - DKIM records
  - SPF record
  - DMARC record
- Create welcome/signup confirmation email template via Resend API
- Scaffold email sending utility in code
- Store sending domain in project.json
```

#### Step 13: Monitoring

```
GCP Uptime Check:
  - Create HTTPS uptime check on {domain} (or Cloud Run URL if no domain yet)
  - Create notification channel (email)
  - Link alert policy to notification channel

Cloud Monitoring Alerts:
  - Alert policy: 5xx error rate > 1% over 5 minutes
  - Alert policy: p95 latency > 2s over 5 minutes
  - Both linked to email notification channel

Error Tracking:
  - Already enabled in PostHog (Step 10)
  - SDK already installed in scaffold
  - No additional setup needed
```

---

### Phase 3: File Generation (Step 14)

#### `project.json` — Source of Truth (committed, no secrets)

```json
{
  "name": "My SaaS App",
  "slug": "my-saas-app",
  "description": "One-line description",
  "type": "saas",
  "created": "2026-03-29",

  "architecture": {
    "language": "typescript",
    "framework": "next.js",
    "database": "cloud-sql-postgres",
    "orm": "drizzle",
    "auth": "firebase-auth",
    "hosting": "cloud-run",
    "cicd": "github-actions",
    "linter": "biome"
  },

  "gcp": {
    "projectId": "my-saas-app",
    "region": "europe-west9",
    "serviceAccount": "my-saas-app-sa@my-saas-app.iam.gserviceaccount.com",
    "cloudRunService": "my-saas-app",
    "dnsZone": "my-saas-app-zone"
  },

  "domain": "mysaasapp.com",

  "github": {
    "org": "daedalium",
    "repo": "my-saas-app",
    "visibility": "private"
  },

  "services": {
    "ga4": {
      "propertyId": "123456789",
      "measurementId": "G-XXXXXXXXXX",
      "dataStreamId": "987654321"
    },
    "searchConsole": {
      "verified": true
    },
    "posthog": {
      "orgId": "org_xxx",
      "projectId": "12345",
      "host": "https://app.posthog.com"
    },
    "stripe": {
      "mode": "test",
      "productId": "prod_xxx",
      "priceId": "price_xxx",
      "webhookEndpointId": "we_xxx",
      "customerPortalEnabled": true
    },
    "resend": {
      "sendingDomain": "mysaasapp.com",
      "domainVerified": true
    },
    "errorTracking": {
      "provider": "posthog",
      "enabled": true
    }
  },

  "monitoring": {
    "uptimeCheck": "my-saas-app-uptime",
    "alertPolicies": ["5xx-rate", "latency"]
  },

  "environments": ["production"]
}
```

#### `PROJECT.md` — Generated from project.json (committed)

Human-readable project card with:
- Project name + description
- Quick links table (GitHub, GCP Console, GA4, PostHog, Stripe, Search Console)
- Architecture summary
- Service status
- Auto-regenerated when `project.json` changes

#### `.env` — Secrets (gitignored)

All API keys, secret keys, webhook secrets. Framework-appropriate variable naming (e.g., `NEXT_PUBLIC_` prefix for client-side values in Next.js).

#### `.env.example` — Template (committed)

Same structure as `.env`, empty values. Documents what env vars the project needs.

#### `CLAUDE.md` — Seeded

Contains:
- Project name, type, description
- Tech stack summary
- GCP project ID and region
- All commands: build, dev, test, deploy, lint
- Location of `project.json`, `.env`, `~/.gcloud-keys/{slug}.json`
- Secret Manager reference for production
- Links to dashboards
- Architecture conventions from the discussion
- Reference to `docs/decisions/` for ADRs

#### `docs/decisions/001-initial-architecture.md` — First ADR

One section per architecture decision with:
- What was chosen
- Alternatives considered
- Rationale (including user's reasoning)

#### `README.md` — Beautiful README

Best practices:
- Custom ASCII art header generated from project name
- One-paragraph description that sells the project
- Badges (build status, license, version)
- Screenshot/demo placeholder
- Quick start (clone → install → run in 3 steps)
- Architecture overview with brief diagram
- Tech stack table
- Development setup
- Deployment instructions
- Contributing guidelines
- License section

The README should feel like a polished open-source project from day one.

#### `.gitignore` — Stack-tailored

Generated based on language + framework chosen during architecture phase. Covers:
- Build artifacts
- Dependencies
- Environment files
- IDE files
- OS files
- Framework-specific ignores

---

### Phase 4: Ship (Steps 15-16)

#### Step 15: First Commit + Push

```
- git add -A
- git commit -m "Initial project setup via /startup"
- git push -u origin main
```

Single commit. Project arrives on GitHub fully formed.

#### Step 16: Summary Report

Structured output showing:
- All resource URLs (GitHub, GCP Console, Cloud Run, domain)
- All dashboard links (GA4, Search Console, PostHog, Stripe)
- Local dev commands (dev, build, test, deploy)
- Manual steps remaining (typically just: paste nameservers in Namecheap)
- Any failed provisioning steps with re-run instructions

---

## Idempotency

`/startup` is safe to re-run. Every provisioning step checks for existing resources before creating. On re-run:
- Existing resources are skipped with "already exists" log
- Missing resources are created
- Files are regenerated (project.json, PROJECT.md, CLAUDE.md updated)
- Summary shows what was created vs. what was skipped

This handles: partial failures, interrupted runs, and "I forgot to add Stripe" scenarios (change project.json, re-run).

---

## GCP Authentication Flow

1. Skill checks `gcloud auth list` for active account
2. If not logged in: prompt user to run `! gcloud auth login`
3. User's personal auth creates the GCP project and service account
4. Service account key is downloaded to `~/.gcloud-keys/{slug}.json`
5. All subsequent operations in deploy scripts use the service account (per existing convention)

---

## Allowed Tools

```yaml
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - AskUserQuestion
  - WebSearch
  - mcp__plugin_posthog_posthog__*    # PostHog project creation
  - mcp__plugin_stripe_stripe__*      # Stripe product/webhook setup
```

The skill uses Bash for `gcloud`, `gh`, and Google API calls. PostHog and Stripe MCP tools for their respective service setup. AskUserQuestion for the interview and architecture conversation.

---

## What This Skill Does NOT Do

- **Brainstorming** — use `/office-hours` for idea validation
- **Retrofitting** — only works on new/empty projects
- **Business logic** — scaffold is a working shell, not a product
- **Ongoing maintenance** — CLAUDE.md auto-update is a separate concern
- **Design system** — use `/design-consultation` after `/startup`
- **Production deployment** — use `/ship` and `/land-and-deploy` for that

---

## Open Questions

None — all questions resolved during brainstorming session.

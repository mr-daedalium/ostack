# Provisioning: GCP Project

**Idempotency:** Check if each resource exists before creating. Skip and log if it does.

---

## Step 1: Verify gcloud authentication

```bash
ACCOUNT=$(gcloud auth list --filter=status:ACTIVE --format="value(account)" 2>/dev/null)
echo "ACTIVE_ACCOUNT: $ACCOUNT"
```

If no active account, prompt the user:
> You need to authenticate with Google Cloud first. Please run:
> `! gcloud auth login`
> Then I'll continue.

Wait for the user to authenticate before proceeding.

## Step 2: Create GCP project

```bash
gcloud projects describe {slug} 2>/dev/null && echo "PROJECT_EXISTS" || echo "PROJECT_MISSING"
```

If missing:
```bash
gcloud projects create {slug} --name="{project-name}"
```

If the project ID is taken (globally unique constraint), inform the user and ask for an alternative slug.

## Step 3: Link billing account

```bash
# Detect billing account (user has one billing account)
BILLING=$(gcloud billing accounts list --filter=open=true --format="value(name)" --limit=1)
echo "BILLING_ACCOUNT: $BILLING"

# Check current billing link
gcloud billing projects describe {slug} --format="value(billingAccountName)" 2>/dev/null
```

If not linked:
```bash
gcloud billing projects link {slug} --billing-account=$BILLING
```

## Step 4: Set active project

```bash
gcloud config set project {slug}
```

## Step 5: Enable APIs

Always-enable (batch):
```bash
gcloud services enable \
  run.googleapis.com \
  cloudbuild.googleapis.com \
  storage.googleapis.com \
  dns.googleapis.com \
  artifactregistry.googleapis.com \
  secretmanager.googleapis.com \
  logging.googleapis.com \
  iam.googleapis.com \
  cloudscheduler.googleapis.com \
  cloudtasks.googleapis.com \
  --project={slug}
```

Conditional APIs — enable based on architecture decisions:
- If Cloud SQL chosen: `sqladmin.googleapis.com`
- If Firestore chosen: `firestore.googleapis.com`
- If GPU/AI workloads: `compute.googleapis.com`, `aiplatform.googleapis.com`
- If Cloud Functions: `cloudfunctions.googleapis.com`
- If Pub/Sub: `pubsub.googleapis.com`
- If Redis/Memorystore: `redis.googleapis.com`
- If BigQuery: `bigquery.googleapis.com`
- If Maps: `maps-backend.googleapis.com`, `places-backend.googleapis.com`
- If GA4 API: `analyticsadmin.googleapis.com`
- If Search Console: `searchconsole.googleapis.com`
- If Cloud Monitoring: `monitoring.googleapis.com`
- If IAP (internal tools): `iap.googleapis.com`

Add conditional APIs to the batch command above.

## Step 6: Create service account

```bash
# Check if SA exists
gcloud iam service-accounts describe {slug}-sa@{slug}.iam.gserviceaccount.com 2>/dev/null && echo "SA_EXISTS" || echo "SA_MISSING"
```

If missing:
```bash
gcloud iam service-accounts create {slug}-sa \
  --display-name="{project-name} Service Account" \
  --project={slug}
```

Grant Owner role:
```bash
gcloud projects add-iam-policy-binding {slug} \
  --member="serviceAccount:{slug}-sa@{slug}.iam.gserviceaccount.com" \
  --role="roles/owner" \
  --condition=None
```

## Step 7: Download service account key

```bash
# Check if key already exists
ls ~/.gcloud-keys/{slug}.json 2>/dev/null && echo "KEY_EXISTS" || echo "KEY_MISSING"
```

If missing:
```bash
mkdir -p ~/.gcloud-keys
gcloud iam service-accounts keys create ~/.gcloud-keys/{slug}.json \
  --iam-account={slug}-sa@{slug}.iam.gserviceaccount.com
chmod 600 ~/.gcloud-keys/{slug}.json
```

## Output

Log what was created/skipped:
- GCP project: created / already existed
- Billing: linked / already linked
- APIs: list of APIs enabled
- Service account: created / already existed
- Key: downloaded to ~/.gcloud-keys/{slug}.json / already existed

# Provisioning: CI/CD Pipeline

Set up the CI/CD pipeline based on the architecture decision.

---

## GitHub Actions (default)

If GitHub Actions was chosen, create `.github/workflows/deploy.yml`:

```yaml
name: Deploy

on:
  push:
    branches: [main]

env:
  PROJECT_ID: {slug}
  REGION: {region}
  SERVICE: {slug}

jobs:
  deploy:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      id-token: write

    steps:
      - uses: actions/checkout@v4

      - id: auth
        uses: google-github-actions/auth@v2
        with:
          credentials_json: ${{ secrets.GCP_SA_KEY }}

      - uses: google-github-actions/setup-gcloud@v2

      - name: Deploy to Cloud Run
        run: |
          gcloud run deploy $SERVICE \
            --source . \
            --region $REGION \
            --project $PROJECT_ID \
            --allow-unauthenticated \
            --quiet
```

**Setup required:**
1. Add `GCP_SA_KEY` as a GitHub Actions secret (base64-encoded service account key)
2. The key is at `~/.gcloud-keys/{slug}.json`

```bash
# Encode the key for GitHub Actions
cat ~/.gcloud-keys/{slug}.json | base64 | gh secret set GCP_SA_KEY --repo={org}/{slug}
```

Also create a lint/test workflow `.github/workflows/ci.yml`:

```yaml
name: CI

on:
  pull_request:
    branches: [main]
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      # Framework-specific setup (bun, node, python, rust, etc.)
      # Lint + test commands from CLAUDE.md
```

Adapt the CI workflow to the chosen framework:
- Next.js / TypeScript → `bun install && bun run lint && bun test`
- Python → `uv sync && ruff check . && pytest`
- Rust → `cargo fmt --check && cargo clippy && cargo test`

## Cloud Build (alternative)

If Cloud Build was chosen instead:

Create `cloudbuild.yaml`:

```yaml
steps:
  - name: 'gcr.io/cloud-builders/gcloud'
    args:
      - 'run'
      - 'deploy'
      - '{slug}'
      - '--source=.'
      - '--region={region}'
      - '--allow-unauthenticated'
      - '--quiet'
```

Set up Cloud Build trigger:
```bash
gcloud builds triggers create github \
  --repo-name={slug} \
  --repo-owner={org} \
  --branch-pattern="^main$" \
  --build-config=cloudbuild.yaml \
  --project={slug}
```

## Output

Log:
- CI/CD: GitHub Actions / Cloud Build configured
- Workflows created: deploy.yml, ci.yml
- GitHub secret: GCP_SA_KEY set (if GitHub Actions)

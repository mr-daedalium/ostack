# Provisioning: Resend (Conditional — Email Projects Only)

**Skip this step if:** Project doesn't need email.

---

## Step 1: Store Resend API key

Ask the user for their Resend API key via AskUserQuestion:
> Enter your Resend API key (from https://resend.com/api-keys):

Store in `.env` as `RESEND_API_KEY`.

## Step 2: Add domain DNS records to Cloud DNS

Resend requires DNS records for domain verification (DKIM, SPF, DMARC). These records are specific to the sending domain.

After the user provides the API key, use the Resend API to get the required DNS records:

```bash
TOKEN="{resend-api-key}"

# Add domain to Resend
curl -s -X POST \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"name": "{domain}"}' \
  "https://api.resend.com/domains"
```

From the response, extract the DNS records and add them to Cloud DNS:

```bash
# Add DKIM records (typically 3 CNAME records)
gcloud dns record-sets create {dkim-host}. \
  --type=CNAME \
  --ttl=300 \
  --rrdatas="{dkim-value}." \
  --zone={slug}-zone \
  --project={slug}

# Add SPF record (TXT)
gcloud dns record-sets create {domain}. \
  --type=TXT \
  --ttl=300 \
  --rrdatas='"v=spf1 include:amazonses.com ~all"' \
  --zone={slug}-zone \
  --project={slug}

# Add DMARC record (TXT)
gcloud dns record-sets create _dmarc.{domain}. \
  --type=TXT \
  --ttl=300 \
  --rrdatas='"v=DMARC1; p=none;"' \
  --zone={slug}-zone \
  --project={slug}
```

**Note:** Exact DNS records come from the Resend API response. The above are examples — use the actual values from the response.

## Step 3: Create welcome email template

Use the Resend API to create a basic welcome email template:

```bash
curl -s -X POST \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "welcome",
    "subject": "Welcome to {project-name}",
    "html": "<h1>Welcome to {project-name}!</h1><p>Thanks for signing up. We'\''re excited to have you.</p>"
  }' \
  "https://api.resend.com/emails"
```

**Note:** Resend may not have a template creation API. In that case, scaffold the template as a code file in the project instead (e.g., `src/emails/welcome.tsx` using React Email).

## Step 4: Store values

- `.env`: `RESEND_API_KEY`
- `project.json` under `services.resend`:
  - `sendingDomain`: the domain
  - `domainVerified`: true/false (check after DNS records are added)

## Output

Log:
- Resend domain: added + DNS records created in Cloud DNS
- Domain verification: pending (DNS propagation)
- Welcome template: created in code / via API

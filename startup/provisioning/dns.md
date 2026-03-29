# Provisioning: Cloud DNS

**Idempotency:** Check if DNS zone exists before creating.

**Skip this step if:** No domain was provided in Phase 1.

---

## Step 1: Check if DNS zone exists

```bash
gcloud dns managed-zones describe {slug}-zone --project={slug} 2>/dev/null && echo "ZONE_EXISTS" || echo "ZONE_MISSING"
```

## Step 2: Create DNS zone

If missing:
```bash
gcloud dns managed-zones create {slug}-zone \
  --dns-name="{domain}." \
  --description="{project-name} DNS zone" \
  --project={slug}
```

## Step 3: Extract nameservers

```bash
gcloud dns managed-zones describe {slug}-zone \
  --project={slug} \
  --format="value(nameServers)"
```

## Step 4: Display nameservers

Present to the user:

> **ACTION REQUIRED — Set nameservers in Namecheap:**
>
> 1. Go to Namecheap → Domain List → {domain} → Custom DNS
> 2. Enter these nameservers:
>    - ns-cloud-a1.googledomains.com
>    - ns-cloud-a2.googledomains.com
>    - ns-cloud-a3.googledomains.com
>    - ns-cloud-a4.googledomains.com
>    (Use the actual nameservers from the output above)
>
> DNS propagation takes 15 minutes to 48 hours.

This is the one expected manual step in the entire /startup flow.

## Output

Log:
- DNS zone: created / already existed
- Nameservers: listed (user must set in Namecheap)

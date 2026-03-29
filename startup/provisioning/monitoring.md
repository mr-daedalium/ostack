# Provisioning: Monitoring (Uptime Checks + Alerts)

---

## Step 1: Enable Cloud Monitoring API

```bash
gcloud services enable monitoring.googleapis.com --project={slug}
```

## Step 2: Create notification channel (email)

```bash
# Get user's email from gcloud auth
EMAIL=$(gcloud auth list --filter=status:ACTIVE --format="value(account)")

# Create email notification channel
gcloud alpha monitoring channels create \
  --type=email \
  --display-name="{project-name} Alerts" \
  --channel-labels=email_address=$EMAIL \
  --project={slug}
```

Extract the channel ID from the output.

**Note:** If `gcloud alpha monitoring channels create` is not available, use the REST API:

```bash
TOKEN=$(gcloud auth print-access-token)
curl -s -X POST \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "type": "email",
    "displayName": "{project-name} Alerts",
    "labels": {"email_address": "'$EMAIL'"}
  }' \
  "https://monitoring.googleapis.com/v3/projects/{slug}/notificationChannels"
```

## Step 3: Create uptime check

```bash
TOKEN=$(gcloud auth print-access-token)

# Target: domain if available, else Cloud Run URL
CHECK_HOST="{domain or cloud-run-url}"

curl -s -X POST \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "displayName": "{slug}-uptime",
    "monitoredResource": {
      "type": "uptime_url",
      "labels": {"host": "'$CHECK_HOST'", "project_id": "{slug}"}
    },
    "httpCheck": {
      "path": "/api/health",
      "port": 443,
      "useSsl": true,
      "requestMethod": "GET"
    },
    "period": "300s",
    "timeout": "10s"
  }' \
  "https://monitoring.googleapis.com/v3/projects/{slug}/uptimeCheckConfigs"
```

## Step 4: Create alert policies

### 5xx Error Rate Alert

```bash
curl -s -X POST \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "displayName": "{slug} 5xx Error Rate",
    "conditions": [{
      "displayName": "5xx rate > 1%",
      "conditionThreshold": {
        "filter": "resource.type=\"cloud_run_revision\" AND resource.labels.service_name=\"{slug}\" AND metric.type=\"run.googleapis.com/request_count\" AND metric.labels.response_code_class=\"5xx\"",
        "comparison": "COMPARISON_GT",
        "thresholdValue": 0.01,
        "duration": "300s",
        "aggregations": [{
          "alignmentPeriod": "300s",
          "perSeriesAligner": "ALIGN_RATE"
        }]
      }
    }],
    "notificationChannels": ["{channel-id}"],
    "combiner": "OR"
  }' \
  "https://monitoring.googleapis.com/v3/projects/{slug}/alertPolicies"
```

### Latency Alert

```bash
curl -s -X POST \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "displayName": "{slug} High Latency",
    "conditions": [{
      "displayName": "p95 latency > 2s",
      "conditionThreshold": {
        "filter": "resource.type=\"cloud_run_revision\" AND resource.labels.service_name=\"{slug}\" AND metric.type=\"run.googleapis.com/request_latencies\"",
        "comparison": "COMPARISON_GT",
        "thresholdValue": 2000,
        "duration": "300s",
        "aggregations": [{
          "alignmentPeriod": "300s",
          "perSeriesAligner": "ALIGN_PERCENTILE_95"
        }]
      }
    }],
    "notificationChannels": ["{channel-id}"],
    "combiner": "OR"
  }' \
  "https://monitoring.googleapis.com/v3/projects/{slug}/alertPolicies"
```

## Output

Log:
- Notification channel: created (email: {email})
- Uptime check: created on {host}/api/health
- Alert policy: 5xx error rate > 1% → email
- Alert policy: p95 latency > 2s → email

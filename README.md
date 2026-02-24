# AlertBridge

AlertBridge is a Jira Cloud app that converts incoming alerts into Jira issues using a secure webhook.  
It enables teams to track, update, and resolve incidents directly within Jira.

---

## Overview

AlertBridge receives alert events via HTTPS webhook and automatically:

- Creates Jira issues when alerts fire
- Deduplicates repeated alerts
- Adds comments for recurring alerts
- Transitions issues when alerts resolve
- Reopens issues if alerts fire again

The app is compatible with any system capable of sending JSON payloads over HTTP.

---

## Installation

1. Install **AlertBridge** from the Atlassian Marketplace
2. Open **Jira Settings → Apps → AlertBridge**
3. Open the **AlertBridge Configuration** page

---

## Configuration

From the AlertBridge administration page:

- Copy the generated **Webhook URL**
- Generate or regenerate the **Bearer Token**
- Select the target **Jira Project**
- Select the **Issue Type**
- (Optional) Configure ignored alert labels
- Click **Save**

---

## Sending a Test Alert

You can test AlertBridge using `curl`.

### Example: Firing Alert

```bash
curl -X POST "YOUR_WEBHOOK_URL" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "status": "firing",
    "labels": {
      "alertname": "TestAlert",
      "severity": "warning"
    },
    "annotations": {
      "summary": "Test alert from AlertBridge"
    }
  }'

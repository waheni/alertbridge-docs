# AlertBridge

AlertBridge is a Jira Cloud app that converts incoming alerts into Jira issues using a secure webhook.

It enables teams to automatically create, update, resolve, and reopen Jira issues based on alert lifecycle events.

---

## Overview

AlertBridge receives alert events via HTTPS webhook and automatically:

- Creates Jira issues when alerts fire
- Deduplicates repeated alerts
- Adds comments for recurring alerts
- Transitions issues when alerts resolve
- Reopens issues if alerts fire again

AlertBridge works with any system capable of sending JSON payloads over HTTP.

---

## Prerequisites

- Jira Administrator permissions are required to configure the app
- The alert source must support HTTPS POST requests
- Payloads must be valid JSON
- Requests must include Bearer token authentication

---

## Installation

1. Install **AlertBridge** from the Atlassian Marketplace
2. Open **Jira Settings → Apps → AlertBridge**
3. Open the **AlertBridge Configuration** page

---

## Configuration

From the AlertBridge administration page:

1. Copy the generated **Webhook URL**
2. Generate or regenerate the **Bearer Token**
3. Select the target **Jira Project**
4. Select the **Issue Type**
5. (Optional) Configure ignored alert labels
6. Click **Save**

---

## Sending a Test Alert

You can validate AlertBridge using `curl`.

---

### Example: Firing Alert

```bash
curl -X POST "YOUR_WEBHOOK_URL" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "status": "firing",
    "labels": {
      "alertname": "AlertBridgeDemo",
      "severity": "warning"
    },
    "annotations": {
      "summary": "Test alert from AlertBridge"
    }
  }'

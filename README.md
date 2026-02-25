# AlertBridge User Guide

**Version:** 1.1.31  
**Last Updated:** February 25, 2026

---

## Table of Contents

1. [Introduction](#introduction)
2. [What is AlertBridge?](#what-is-alertbridge)
3. [Key Features](#key-features)
4. [Getting Started](#getting-started)
5. [Configuration Guide](#configuration-guide)
6. [How to Use AlertBridge](#how-to-use-alertbridge)
7. [Use Cases and Examples](#use-cases-and-examples)
8. [Understanding Alert Processing](#understanding-alert-processing)
9. [Security](#security)
10. [Limitations](#limitations)
11. [Troubleshooting](#troubleshooting)
12. [Frequently Asked Questions](#frequently-asked-questions)

---

## Introduction

AlertBridge is a Jira Cloud app that automatically creates and manages Jira issues from monitoring alerts. It acts as a bridge between your monitoring systems (like Prometheus, Grafana, or any custom alerting tool) and Jira, ensuring that critical alerts are tracked as actionable issues in your project management workflow.

This guide is designed for end users who need to configure and use AlertBridge to integrate their alerting systems with Jira.

---

## What is AlertBridge?

AlertBridge receives alert notifications from external monitoring systems via webhooks and automatically:

- **Creates Jira issues** when alerts fire
- **Updates existing issues** when alerts change or re-fire
- **Adds comments** to issues with alert status updates
- **Resolves issues** automatically when alerts are cleared
- **Prevents duplicate issues** by tracking alert fingerprints

### Why Use AlertBridge?

**Problem:** Your monitoring system generates alerts, but they're scattered across different tools, making it hard to track who's responsible for fixing issues, what the status is, and whether problems were resolved.

**Solution:** AlertBridge brings all alerts into Jira as issues, giving you:
- **Centralized tracking** of all incidents
- **Assignment and ownership** through Jira's workflow
- **Historical records** of all alerts and resolutions
- **Integration** with your existing project management processes

---

## Key Features

### 🔔 Automatic Issue Creation
When an alert fires, AlertBridge automatically creates a Jira issue with:
- Alert name and severity in the title
- Detailed description including all alert labels
- Link to the alert source (generator URL)
- Priority based on alert severity

### 🔄 Smart Deduplication
AlertBridge uses alert fingerprints to prevent creating duplicate issues. If the same alert fires multiple times, it updates the existing issue instead of creating new ones.

### 📝 Automatic Updates
- Adds comments when alerts re-fire
- Tracks how many times an alert has triggered
- Updates timestamps for last occurrence

### ✅ Auto-Resolution
When an alert is resolved in your monitoring system, AlertBridge automatically:
- Transitions the Jira issue to "Done" or "Resolved"
- Adds a comment indicating the alert is cleared
- Keeps the issue history for reference

### 🔁 Auto-Reopening
If a previously resolved alert fires again, AlertBridge can automatically reopen the issue instead of creating a new one.

### 🔒 Secure Authentication
All webhook requests are authenticated using Bearer tokens, ensuring only authorized systems can create issues.

### 🎯 Severity Mapping
Automatically maps alert severity levels to Jira priorities:
- `critical` → Highest
- `high` → High
- `medium` → Medium
- `low` → Low

---

## Getting Started

### Prerequisites

Before using AlertBridge, ensure you have:

1. **Jira Cloud** site with administrative access
2. **AlertBridge app installed** in your Jira instance
3. **A monitoring system** that can send webhook notifications (Prometheus Alertmanager, Grafana, etc.)
4. **Permission** to create issues in your target Jira project

### Initial Setup Overview

The setup process involves three main steps:

1. **Access the AlertBridge configuration page** in Jira
2. **Generate a webhook token** for authentication
3. **Configure your target project and issue type**
4. **Connect your monitoring system** to AlertBridge

---

## Configuration Guide

### Step 1: Access AlertBridge Configuration

1. Log in to your **Jira Cloud** instance
2. Click on **⚙️ Settings** (gear icon) in the top right
3. Navigate to **Apps** → **Manage Apps**
4. Find **AlertBridge** in the list
5. Click on **AlertBridge** to open the configuration page

Alternatively:
- Use the Jira navigation menu and look for **AlertBridge** under Apps or Admin pages

### Step 2: Generate Webhook Token

The webhook token is a secure password that authenticates requests from your monitoring system.

1. On the AlertBridge configuration page, locate the **"Webhook URL & Token"** section
2. Click the **"Generate Token"** button
3. **⚠️ IMPORTANT:** Copy the token immediately and store it securely
   - The full token is shown **only once** when generated
   - After you refresh the page, only a masked version (e.g., `a3f2****b9c4`) will be visible
   - If you lose the token, you'll need to regenerate it

**Example Token:**
```
a3f27b8c9e1d4f6a8b2c5d7e9f1a3b4c
```

### Step 3: Copy the Webhook URL

Below the token section, you'll see the **Webhook URL**. This is the address your monitoring system will send alerts to.

**Example Webhook URL:**
```
https://api.atlassian.com/webhook/alertbridge-webhook-xxxxxxxxxxxx
```

Copy this URL - you'll need it when configuring your monitoring system.

### Step 4: Configure Jira Target

This tells AlertBridge where to create issues.

1. In the **"Jira Target"** section, select:
   - **Project:** Choose the Jira project where alert issues should be created
   - **Issue Type:** Select the type of issue (e.g., "Bug", "Task", "Incident")

2. **(Optional) Configure Label Filtering:**
   - In the **"Ignore label keys"** field, enter comma-separated label names to exclude from fingerprint calculation
   - **Default:** `instance,pod,container,node`
   - **Why?** These labels often change frequently (like pod IDs) but represent the same alert. Ignoring them prevents creating duplicate issues.

**Example Configuration:**
- Project: `OPERATIONS`
- Issue Type: `Incident`
- Ignore label keys: `instance,pod,container,node`

3. Click **"Save Configuration"** button

### Step 5: Advanced Configuration (Optional)

AlertBridge uses default settings that work for most users, but you can customize:

#### Severity to Priority Mapping
AlertBridge automatically maps alert severities to Jira priorities:
- `critical` → Highest
- `high` → High
- `medium` → Medium  
- `low` → Low

This mapping is currently handled automatically. If an alert has a severity label, the corresponding priority will be set on the Jira issue.

#### Transition Names
AlertBridge tries to automatically transition issues when alerts resolve or fire again:
- **Resolve Transition:** Default is "Done" (also tries "Resolved", "Close", "Closed")
- **Reopen Transition:** Default is "Reopen" (also tries "To Do", "Open")

These transitions must exist in your Jira workflow. If they don't, AlertBridge will add comments instead.

---

## How to Use AlertBridge

### Connecting Your Monitoring System

AlertBridge works with any monitoring system that can send HTTP POST requests. Below are instructions for common tools:

#### Option 1: Prometheus Alertmanager

1. Edit your Alertmanager configuration file (`alertmanager.yml`)
2. Add a webhook receiver:

```yaml
receivers:
  - name: 'jira-alertbridge'
    webhook_configs:
      - url: 'https://api.atlassian.com/webhook/alertbridge-webhook-xxxxxxxxxxxx'
        send_resolved: true
        headers:
          Authorization: 'Bearer a3f27b8c9e1d4f6a8b2c5d7e9f1a3b4c'
          Content-Type: 'application/json'
```

3. Configure your alert routes to use this receiver:

```yaml
route:
  receiver: 'jira-alertbridge'
  routes:
    - match:
        severity: critical
      receiver: 'jira-alertbridge'
```

4. Reload Alertmanager configuration:
```bash
curl -X POST http://localhost:9093/-/reload
```

#### Option 2: Grafana

1. In Grafana, go to **Alerting** → **Contact points**
2. Click **"New contact point"**
3. Set:
   - **Name:** JiraAlertBridge
   - **Type:** webhook
   - **URL:** Your AlertBridge webhook URL
   - **HTTP Method:** POST
4. Add custom header:
   - **Header:** `Authorization`
   - **Value:** `Bearer YOUR_TOKEN_HERE`
5. Save the contact point
6. Assign this contact point to your alert rules

#### Option 3: Custom Scripts or Other Tools

Any system that can send HTTP POST requests can integrate with AlertBridge. The request must:

1. Use POST method
2. Include the `Authorization: Bearer YOUR_TOKEN` header
3. Send JSON in Alertmanager webhook format (see [Alert Payload Format](#alert-payload-format))

**Example using curl:**
```bash
curl -X POST "https://api.atlassian.com/webhook/alertbridge-webhook-xxxxxxxxxxxx" \
  -H "Authorization: Bearer a3f27b8c9e1d4f6a8b2c5d7e9f1a3b4c" \
  -H "Content-Type: application/json" \
  -d '{
    "status": "firing",
    "alerts": [{
      "status": "firing",
      "labels": {
        "alertname": "HighCPUUsage",
        "severity": "critical",
        "service": "web-server",
        "env": "production"
      },
      "annotations": {
        "summary": "CPU usage above 90%",
        "description": "The web server CPU usage has been above 90% for 5 minutes"
      },
      "generatorURL": "https://prometheus.example.com/graph?g0.expr=..."
    }]
  }'
```

### Testing Your Setup

After configuration, test that everything works:

1. On the AlertBridge configuration page, find the **"Test with curl"** section
2. Copy the curl command template
3. Replace `<WEBHOOK_URL>` and `<YOUR_TOKEN>` with your actual values
4. Run the command in a terminal:

```bash
curl -i -X POST "https://api.atlassian.com/webhook/alertbridge-webhook-xxxxxxxxxxxx" \
  -H "Authorization: Bearer a3f27b8c9e1d4f6a8b2c5d7e9f1a3b4c" \
  -H "Content-Type: application/json" \
  --data-binary '{
    "status":"firing",
    "alerts":[{
      "status":"firing",
      "labels":{
        "alertname":"TestAlert",
        "severity":"critical",
        "service":"test-service"
      },
      "annotations":{
        "summary":"This is a test alert"
      },
      "generatorURL":"https://example.com"
    }]
  }'
```

5. **Expected Result:** 
   - HTTP 200 response
   - A new issue created in your configured Jira project
   - Issue title: `[critical] TestAlert - test-service`

6. Check the **"Webhook Status"** section on the AlertBridge page:
   - **Last webhook:** Should show current timestamp
   - **Created:** Should increment by 1
   - **Last error:** Should show `-` (no errors)

### Monitoring Webhook Activity

The AlertBridge configuration page shows real-time webhook activity:

- **Last webhook:** Timestamp of the most recent webhook received
- **Last error:** Most recent error message (if any)
- **Created:** Total number of issues created since installation
- **Updated:** Total number of times existing issues were updated

Click the **"Refresh"** button to update these statistics.

---

## Use Cases and Examples

### Use Case 1: Production Infrastructure Monitoring

**Scenario:** You run a production web application and use Prometheus to monitor server health, CPU, memory, and application errors.

**Setup:**
1. Configure AlertBridge with Project: `DEVOPS`, Issue Type: `Incident`
2. Set ignore labels: `instance,pod,node` (since these change with autoscaling)
3. Connect Prometheus Alertmanager to AlertBridge

**Alert Example 1 - High CPU:**
```json
{
  "status": "firing",
  "alerts": [{
    "status": "firing",
    "labels": {
      "alertname": "HighCPUUsage",
      "severity": "critical",
      "service": "api-server",
      "env": "production",
      "instance": "10.0.1.45:9090"
    },
    "annotations": {
      "summary": "CPU usage above 90% for 5 minutes",
      "description": "Production API server CPU is at 95%"
    }
  }]
}
```

**Result in Jira:**
- **Issue Title:** `[critical] HighCPUUsage - api-server (production)`
- **Priority:** Highest
- **Description:** Contains the summary, description, all labels, and source link
- **Status:** Open

**What happens next:**
1. If the same alert fires again (same service, same alertname), AlertBridge adds a comment like: `🔁 Alert update (#2): [critical] HighCPUUsage...`
2. When CPU returns to normal, Prometheus sends a "resolved" webhook
3. AlertBridge transitions the issue to "Done" and adds comment: `✅ Resolved (auto-transition): [critical] HighCPUUsage...`

### Use Case 2: Application Error Monitoring

**Scenario:** Your application logs errors to a monitoring system, and you want critical errors to create Jira issues for investigation.

**Alert Example - Application Error:**
```json
{
  "status": "firing",
  "alerts": [{
    "status": "firing",
    "labels": {
      "alertname": "DatabaseConnectionFailed",
      "severity": "high",
      "service": "payment-service",
      "env": "production",
      "error_type": "connection_timeout"
    },
    "annotations": {
      "summary": "Unable to connect to payment database",
      "description": "Payment service cannot reach the database. Last error: connection timeout after 30s"
    },
    "generatorURL": "https://grafana.example.com/d/errors"
  }]
}
```

**Result in Jira:**
- **Issue Title:** `[high] DatabaseConnectionFailed - payment-service (production)`
- **Priority:** High
- **Description:** Full error details with labels and Grafana link
- **Assignee:** Can be assigned based on your Jira automation rules

### Use Case 3: Website Uptime Monitoring

**Scenario:** You use an external monitoring service to check if your website is accessible. When it's down, you want an immediate Jira issue.

**Alert Example - Site Down:**
```json
{
  "status": "firing",
  "alerts": [{
    "status": "firing",
    "labels": {
      "alertname": "WebsiteDown",
      "severity": "critical",
      "service": "company-website",
      "url": "https://www.example.com"
    },
    "annotations": {
      "summary": "Website is unreachable",
      "description": "HTTPS check failed with status 503"
    }
  }]
}
```

**Result:**
- Immediate Jira issue with Highest priority
- Team is notified via Jira notifications
- Issue is automatically resolved when the site comes back online

### Use Case 4: Handling Alert Flapping

**Scenario:** Sometimes alerts flip between firing and resolved rapidly ("flapping"). AlertBridge handles this gracefully.

**Timeline:**
1. **10:00 AM** - Alert fires → Issue created: `DEVOPS-123`
2. **10:05 AM** - Alert resolves → Issue transitioned to "Done"
3. **10:10 AM** - Alert fires again → Issue DEVOPS-123 reopened (not a new issue created)
4. **10:15 AM** - Alert fires again (still firing) → Comment added: "Alert update (#2)"
5. **10:30 AM** - Alert finally resolves → Issue transitioned to "Done" again

**Benefit:** You have a single issue (DEVOPS-123) with the complete history, rather than 3 separate issues.

---

## Understanding Alert Processing

### Alert Payload Format

AlertBridge expects alerts in the **Prometheus Alertmanager webhook format**. Here's the structure:

```json
{
  "status": "firing|resolved",
  "alerts": [
    {
      "status": "firing|resolved",
      "labels": {
        "alertname": "AlertName",
        "severity": "critical|high|medium|low",
        "service": "service-name",
        "env": "production",
        "...": "any custom labels"
      },
      "annotations": {
        "summary": "Brief description",
        "description": "Detailed explanation",
        "...": "any custom annotations"
      },
      "startsAt": "2026-02-25T10:00:00Z",
      "endsAt": "0001-01-01T00:00:00Z",
      "generatorURL": "https://monitoring-tool.example.com/...",
      "fingerprint": "a1b2c3d4e5f6"
    }
  ]
}
```

**Key Fields:**

- **`status`**: Overall status of the alert group (firing or resolved)
- **`alerts[]`**: Array of individual alerts
- **`labels`**: Key-value pairs identifying the alert
  - `alertname`: Name of the alert (required)
  - `severity`: Severity level (optional, but recommended)
  - Custom labels: service, env, region, etc.
- **`annotations`**: Human-readable information
  - `summary`: Brief description
  - `description`: Detailed explanation
- **`generatorURL`**: Link back to the monitoring system
- **`fingerprint`**: Unique identifier (optional, AlertBridge calculates if not provided)

### How AlertBridge Processes Alerts

#### 1. Authentication
- Verifies the `Authorization: Bearer TOKEN` header matches the stored token
- Rejects requests with invalid or missing tokens (HTTP 401)

#### 2. Fingerprint Calculation
AlertBridge computes a unique fingerprint for each alert based on its labels. This fingerprint is used to:
- Detect duplicate alerts
- Link multiple occurrences to the same Jira issue
- Prevent creating redundant issues

**Fingerprint Formula:**
```
SHA-256 hash of sorted labels (excluding ignored labels)
```

**Example:**
If your alert has labels:
```json
{
  "alertname": "HighCPU",
  "service": "api",
  "instance": "pod-abc-123",
  "severity": "critical"
}
```

And you configured ignore labels: `instance,pod`

The fingerprint is calculated from:
```
alertname=HighCPU|service=api|severity=critical
```

Two alerts with the same fingerprint update the same Jira issue, even if the ignored labels differ.

#### 3. Issue Creation or Update

**For FIRING alerts:**

- **If no issue exists with this fingerprint:**
  - Create a new Jira issue
  - Set title: `[severity] alertname - service (env)`
  - Set priority based on severity
  - Add description with all labels and annotations
  - Store fingerprint → issue mapping

- **If issue exists and is OPEN:**
  - Add a comment: `🔁 Alert update (#N)`
  - Increment occurrence counter
  - Update last seen timestamp

- **If issue exists but is RESOLVED:**
  - Try to transition issue to "Reopen" (or similar)
  - Add a comment: `🔥 Alert firing again`
  - Mark issue as open again

**For RESOLVED alerts:**

- **If issue exists and is OPEN:**
  - Try to transition issue to "Done" (or similar)
  - Add a comment: `✅ Resolved (auto-transition)`
  - Mark issue as closed in AlertBridge tracking

- **If issue doesn't exist or is already closed:**
  - No action taken (alert resolved before we saw it firing)

#### 4. Error Handling

If errors occur (invalid config, Jira API failures, etc.):
- Error is logged to AlertBridge health status
- HTTP 500 returned to caller
- Admin can check "Last error" on the configuration page

---

## Security

### Authentication and Authorization

#### Webhook Token Security
- **Token Storage:** Tokens are stored encrypted in Forge Key-Value Storage (KVS) secrets
- **Token Display:** Full token is shown **only once** when generated; afterwards only a masked version is visible
- **Token Validation:** Every webhook request must include the correct Bearer token in the `Authorization` header
- **Case-Insensitive Headers:** AlertBridge handles headers in a case-insensitive manner to prevent spoofing attempts

#### Best Practices
1. **Treat tokens like passwords:** Never commit them to version control or share them publicly
2. **Rotate tokens regularly:** Regenerate tokens periodically (quarterly or after team member departures)
3. **Use HTTPS only:** Always use the HTTPS webhook URL (provided by default)
4. **Limit access:** Only share the token with authorized monitoring systems
5. **Regenerate if compromised:** If a token is leaked, immediately regenerate it and update your monitoring systems

### Jira Permissions

AlertBridge uses **app-level authorization** to create and update Jira issues. This means:

- **Permissions:** AlertBridge has the scopes:
  - `read:jira-work` - Read Jira issues and projects
  - `write:jira-work` - Create and update issues
  - `storage:app` - Store configuration data

- **No user impersonation:** AlertBridge acts as the app itself, not as a specific user

- **Project permissions:** AlertBridge can only create issues in projects where the app has been granted access

### Data Privacy

- **What data is stored:**
  - Configuration: project key, issue type ID, severity mapping
  - Webhook token (encrypted)
  - Alert fingerprints and Jira issue mappings
  - Health statistics (webhook count, timestamps, errors)

- **What data is NOT stored:**
  - Alert payload content (processed in memory only)
  - User credentials
  - Sensitive monitoring data

- **Data location:** All data is stored in Atlassian's Forge infrastructure (hosted in AWS)

### Network Security

- **Inbound:** Webhook endpoint is public but requires authentication
- **Outbound:** AlertBridge only communicates with Jira REST APIs (no external API calls)
- **No egress:** AlertBridge does not send data to external services

---

## Limitations

### Functional Limitations

#### 1. Single Project per Configuration
- AlertBridge creates all issues in **one project** and **one issue type**
- **Workaround:** Install multiple instances of AlertBridge (not currently supported) or use Jira automation to move issues based on labels

#### 2. Issue Type Restrictions
- All alerts create the same issue type
- Cannot create different issue types based on alert severity or labels
- **Workaround:** Use Jira automation rules to change issue types after creation

#### 3. Workflow Transitions
- AlertBridge attempts to transition issues but depends on your Jira workflow
- **Limitations:**
  - If "Done" or "Reopen" transitions don't exist, AlertBridge adds comments instead
  - Cannot handle complex workflow conditions (e.g., required fields, validators)
  - Transitions with the same name but different IDs may not work correctly
- **Workaround:** Ensure your workflow has standard transition names: "Done", "Reopen"

#### 4. Field Customization
- AlertBridge sets only these fields:
  - Summary (title)
  - Description
  - Priority (based on severity)
  - Project and Issue Type
- **Cannot set:**
  - Assignee
  - Custom fields
  - Labels
  - Components
  - Fix versions
- **Workaround:** Use Jira automation to set additional fields based on issue content

#### 5. Fingerprint Calculation
- Fingerprints are based on **labels only**, not annotations or other alert data
- If labels change (but represent the same alert), a new issue is created
- **Workaround:** Configure ignore labels for frequently-changing label keys (instance, pod, node)

#### 6. Alert Payload Format
- Only supports **Prometheus Alertmanager webhook format**
- Other formats require translation/adaptation
- **Workaround:** Use an intermediate webhook proxy to transform payloads

### Technical Limitations

#### 1. Rate Limits
- Subject to Forge platform rate limits (typically 100 requests per minute)
- Subject to Jira API rate limits
- **Impact:** During alert storms, some requests may be delayed or rejected

#### 2. Processing Time
- Webhook processing is synchronous (caller waits for Jira API calls to complete)
- Typical response time: 1-3 seconds per alert
- **Impact:** Very large alert batches may time out

#### 3. Batch Size
- Processes all alerts in a single webhook request sequentially
- No practical limit on alerts per request, but large batches increase processing time
- **Recommendation:** Keep batches under 50 alerts

#### 4. Storage Limits
- Forge KVS storage is limited per app (typically 10,000 entries)
- Each unique alert fingerprint consumes one storage entry
- **Impact:** After creating ~10,000 unique alerts, storage may be exhausted
- **Workaround:** Periodic cleanup of old fingerprints (manual process)

#### 5. No Alert History
- AlertBridge does not store alert history beyond the current issue mapping
- Cannot query historical alerts through AlertBridge
- **Workaround:** Use Jira's native search and reporting for issue history

### Deployment Limitations

#### 1. Single Environment Configuration
- Configuration is environment-specific (development, staging, production)
- **Impact:** Different environments require separate configuration

#### 2. No Multi-Site Support
- Each Jira site requires a separate AlertBridge installation
- Configuration does not sync across sites

#### 3. Development Mode Token Exposure
- In development mode, the `getFullTokenDev` resolver exposes the full token
- **⚠️ IMPORTANT:** This resolver should be removed before production deployment
- **Impact:** Potential security risk if development mode is used in production

---

## Troubleshooting

### Issue: Webhook Returns 401 Unauthorized

**Symptoms:**
- Your monitoring system shows failed webhook requests
- HTTP 401 response
- AlertBridge health shows "Last error: Unauthorized"

**Causes:**
1. Token mismatch between AlertBridge and monitoring system
2. Token not included in request
3. Incorrect header format

**Solutions:**
1. **Verify token in AlertBridge:**
   - Go to AlertBridge configuration page
   - Check that a token exists (masked token is shown)
   - If unsure, regenerate the token

2. **Check monitoring system configuration:**
   - Verify the `Authorization` header is set to: `Bearer YOUR_TOKEN_HERE`
   - Ensure there's a space between "Bearer" and the token
   - No extra quotes or characters

3. **Test with curl:**
   ```bash
   curl -i -X POST "YOUR_WEBHOOK_URL" \
     -H "Authorization: Bearer YOUR_TOKEN" \
     -H "Content-Type: application/json" \
     -d '{"status":"firing","alerts":[{"status":"firing","labels":{"alertname":"Test"}}]}'
   ```
   Should return HTTP 200

4. **Regenerate token:**
   - Click "Regenerate Token" in AlertBridge
   - Copy the new token immediately
   - Update your monitoring system with the new token

---

### Issue: No Jira Issue Created

**Symptoms:**
- Webhook returns HTTP 200
- No error in AlertBridge health
- But no Jira issue appears

**Causes:**
1. Configuration not saved
2. Invalid project key or issue type ID
3. Jira permissions issue

**Solutions:**
1. **Verify configuration:**
   - Go to AlertBridge configuration page
   - Check that Project and Issue Type are selected
   - Click "Save Configuration" again to ensure it's saved

2. **Check Jira project:**
   - Verify the project key exists and is accessible
   - Try creating an issue manually in that project with that issue type
   - Ensure the issue type is enabled for the project

3. **Check webhook status:**
   - Click "Refresh" on the webhook status section
   - If "Created" count doesn't increment, there's a processing issue
   - Check "Last error" for details

4. **Review alert payload:**
   - Ensure your alert includes required fields:
     - `status: "firing"`
     - `alerts[]` array with at least one alert
     - Each alert has `labels.alertname`

---

### Issue: Duplicate Issues Created

**Symptoms:**
- Multiple Jira issues for what appears to be the same alert
- AlertBridge creates new issues instead of updating existing ones

**Causes:**
1. Alert labels change between firings
2. Fingerprint not matching due to label differences
3. Ignored labels not configured correctly

**Solutions:**
1. **Check which labels are changing:**
   - Compare the labels in the duplicate issues
   - Look for labels like `instance`, `pod`, `container`, `node` that change frequently

2. **Configure ignore labels:**
   - Go to AlertBridge configuration
   - In "Ignore label keys" field, add comma-separated labels that change but shouldn't affect deduplication
   - Example: `instance,pod,container,node,hostname`
   - Click "Save Configuration"

3. **Verify fingerprint consistency:**
   - Ensure your monitoring system sends consistent label keys
   - Avoid timestamps or random IDs in labels

4. **Use provided fingerprint:**
   - If your monitoring system provides a `fingerprint` field, AlertBridge uses it directly
   - Ensure the fingerprint is consistent across alert firings

---

### Issue: Issues Not Transitioning to Done

**Symptoms:**
- Resolved alerts add comments like "Resolved (transition not available)"
- Issues stay open even when alerts are resolved

**Causes:**
1. Workflow transition "Done" doesn't exist
2. Transition has a different name
3. Transition has conditions that aren't met

**Solutions:**
1. **Check workflow transitions:**
   - Go to Jira Project Settings → Workflows
   - Select the workflow for your issue type
   - View available transitions from the "Open" or "In Progress" status
   - Note the exact transition name

2. **Verify transition name:**
   - Common names AlertBridge tries: "Done", "Resolved", "Close", "Closed"
   - If your transition has a different name, issues won't auto-transition

3. **Check transition conditions:**
   - Click on the transition in workflow editor
   - Check if there are conditions, validators, or required fields
   - AlertBridge cannot satisfy complex conditions

4. **Simplify workflow (if possible):**
   - Add a simple "Done" transition without conditions
   - Or remove conditions from the existing close transition

5. **Alternative approach:**
   - Use Jira automation to watch for the "✅ Resolved" comment
   - Automatically transition the issue when that comment appears

---

### Issue: Alerts Not Reopening Resolved Issues

**Symptoms:**
- When an alert fires again after being resolved, a new issue is created instead of reopening the old one
- Or the old issue isn't reopened (no transition occurs)

**Causes:**
1. Reopen transition doesn't exist in workflow
2. Issue was deleted or moved
3. Fingerprint has changed

**Solutions:**
1. **Check workflow for reopen transition:**
   - Verify your workflow has a transition named "Reopen" from "Done" to "Open"
   - AlertBridge also tries: "To Do", "Open"

2. **Add reopen transition:**
   - Edit your workflow
   - Add a transition from "Done" status back to "Open" or "To Do"
   - Name it "Reopen"

3. **Check issue status:**
   - Search for the issue key in Jira
   - If it was deleted, AlertBridge creates a new issue

---

### Issue: Priority Not Set Correctly

**Symptoms:**
- Issues created with wrong priority
- Priority not set at all

**Causes:**
1. Severity label missing or misspelled
2. Priority name doesn't exist in Jira
3. Issue type doesn't support priorities

**Solutions:**
1. **Verify severity label:**
   - Check that alerts include `labels.severity` with values: `critical`, `high`, `medium`, or `low`
   - Label name must be exactly `severity` (lowercase)

2. **Check Jira priorities:**
   - Go to Jira Settings → Issues → Priorities
   - Verify these priorities exist: "Highest", "High", "Medium", "Low"
   - Names must match exactly (case-sensitive)

3. **Check issue type configuration:**
   - Some issue types don't use priorities
   - Try changing to a different issue type (e.g., "Bug" or "Task")

4. **Fallback behavior:**
   - If priority cannot be set, AlertBridge creates the issue without a priority
   - The issue is still created successfully

---

### Issue: Webhook Timeout or Slow Response

**Symptoms:**
- Webhook requests take more than 30 seconds
- Monitoring system reports timeout errors
- AlertBridge returns HTTP 500

**Causes:**
1. Large batch of alerts (50+ alerts)
2. Jira API slow or rate-limited
3. Network issues

**Solutions:**
1. **Reduce batch size:**
   - Configure your monitoring system to send smaller batches
   - Limit to 10-20 alerts per webhook request

2. **Check Jira status:**
   - Visit status.atlassian.com
   - Check if Jira Cloud is experiencing issues

3. **Retry failed requests:**
   - Configure your monitoring system to retry failed webhooks
   - Add exponential backoff (wait longer between retries)

4. **Check webhook health:**
   - Review "Last error" in AlertBridge
   - May provide specific error details

---

### Issue: Token Lost or Forgotten

**Symptoms:**
- Full token not visible on AlertBridge page (only masked)
- Need to reconfigure monitoring system

**Solution:**
1. **Regenerate the token:**
   - Click "Regenerate Token" button
   - Copy the new full token immediately
   - Update your monitoring system configuration with the new token

2. **Note:** You cannot retrieve the old token; it must be regenerated

---

### Issue: Configuration Not Saving

**Symptoms:**
- Click "Save Configuration" but settings revert after page refresh
- Error message appears

**Causes:**
1. Project or issue type not selected
2. Network interruption
3. Permission issue

**Solutions:**
1. **Verify selections:**
   - Ensure both Project AND Issue Type are selected
   - Both fields are required

2. **Check for error messages:**
   - Look for red error banners at the top of the page
   - Error message may indicate the specific problem

3. **Refresh and retry:**
   - Refresh the page
   - Reselect project and issue type
   - Click "Save Configuration" again

4. **Check Jira admin permissions:**
   - Ensure you have admin rights to the Jira site
   - Only admins can configure Forge apps

---

### Issue: Health Status Shows Old Data

**Symptoms:**
- "Last webhook" timestamp is old
- Webhook counts not incrementing

**Solutions:**
1. **Click "Refresh" button:**
   - Health status is cached and doesn't auto-update
   - Click the "Refresh" button in the Webhook Status section

2. **Verify webhooks are being received:**
   - Test with curl command
   - Check your monitoring system's webhook logs

---

### Debugging Tips

1. **Use the test curl command:**
   - Always test with the curl command from the AlertBridge page first
   - This isolates whether the issue is with AlertBridge or your monitoring system

2. **Check webhook status frequently:**
   - The "Webhook Status" section provides real-time feedback
   - "Last error" field is especially useful for diagnosing issues

3. **Verify Jira issue creation manually:**
   - Try creating an issue manually in your target project with the selected issue type
   - This confirms Jira itself is working and you have permissions

4. **Check Jira workflow:**
   - Review your project's workflow to understand available transitions
   - Ensure "Done" and "Reopen" transitions exist for auto-resolution

5. **Review alert payload:**
   - Log or print the webhook payload from your monitoring system
   - Ensure it matches the expected Alertmanager format

6. **Contact support:**
   - If all else fails, contact Atlassian Support with:
     - AlertBridge version
     - Error messages from "Last error"
     - Example webhook payload
     - Steps you've tried

---

## Frequently Asked Questions

### General Questions

**Q: Do I need Prometheus to use AlertBridge?**  
A: No. AlertBridge accepts any HTTP POST request in the Prometheus Alertmanager webhook format. You can use Grafana, custom scripts, or any monitoring tool that can send webhooks.

**Q: Can I use AlertBridge with multiple Jira projects?**  
A: No, currently AlertBridge creates all issues in a single project. As a workaround, use Jira automation rules to move issues to different projects based on their content.

**Q: Does AlertBridge support custom fields?**  
A: No, AlertBridge only populates standard fields (summary, description, priority). Use Jira automation to populate custom fields after issue creation.

**Q: Can I customize the issue summary format?**  
A: No, the summary format is fixed as `[severity] alertname - service (env)`. However, you can use Jira automation to modify issue summaries after creation.

**Q: How many alerts can AlertBridge handle?**  
A: AlertBridge can process hundreds of alerts per minute, but is subject to Forge platform and Jira API rate limits. For very high volumes, consider batching or filtering alerts before sending to AlertBridge.

### Security Questions

**Q: Is the webhook token secure?**  
A: Yes, tokens are stored encrypted in Forge Key-Value Storage and transmitted over HTTPS. Treat tokens like passwords.

**Q: Can I use the same token for multiple monitoring systems?**  
A: Yes, multiple systems can share the same token. However, for better security and audit trails, consider using separate AlertBridge installations for different systems.

**Q: What happens if someone obtains my webhook token?**  
A: They could create fake alert issues in your Jira project. Immediately regenerate the token and update your monitoring systems.

**Q: Does AlertBridge access my monitoring system?**  
A: No, AlertBridge only receives webhooks. It does not make outbound connections to your monitoring system.

### Alert Processing Questions

**Q: Why are multiple alerts updating the same issue?**  
A: AlertBridge uses alert fingerprints (based on labels) to deduplicate. If two alerts have the same labels (excluding ignored labels), they update the same issue.

**Q: How do I prevent duplicate issues?**  
A: Configure "ignore label keys" to exclude labels that change frequently but don't represent different alerts (like `instance`, `pod`, `node`).

**Q: What happens if an alert resolves before I see it?**  
A: If Alertmanager sends a "resolved" webhook before a "firing" webhook, AlertBridge does nothing (no issue created).

**Q: Can I manually resolve issues without waiting for alerts to clear?**  
A: Yes, you can manually transition issues in Jira. AlertBridge will respect the manual state change. If the alert fires again, it will add a comment but not reopen the issue (unless you move it to a resolved state first).

**Q: Do annotations affect deduplication?**  
A: No, only labels affect the fingerprint. Annotations (summary, description) can change without creating a new issue.

### Configuration Questions

**Q: Why can't I see my full token?**  
A: For security, the full token is only shown once when generated. After that, only a masked version is visible. Regenerate if you need to see it again.

**Q: Can I change the severity-to-priority mapping?**  
A: Currently, the mapping is hardcoded. A future version may allow customization. As a workaround, use Jira automation to change priorities based on alert content.

**Q: What if my workflow doesn't have a "Done" transition?**  
A: AlertBridge will try common transition names ("Resolved", "Close", "Closed"). If none exist, it adds a comment instead of transitioning.

**Q: Can I customize which transition is used for resolving alerts?**  
A: The code supports a `resolveTransitionName` configuration parameter, though it may not be exposed in the UI. Check with your Forge app administrator about setting this value directly in configuration storage.

### Troubleshooting Questions

**Q: Why does AlertBridge show "Webhook URL not available"?**  
A: This occurs during development. The webhook URL is generated by Forge and should appear after deployment. If it persists, redeploy the app.

**Q: What does "Config missing" error mean?**  
A: You haven't saved your Project and Issue Type configuration. Go to AlertBridge configuration page and save your settings.

**Q: Why don't I see any issues created even though webhook returns 200?**  
A: Check the "Webhook Status" section for errors. Verify your project key and issue type are valid. Review the "Health" statistics to see if "Created" count increments.

**Q: Can I get notified when AlertBridge encounters errors?**  
A: Not directly from AlertBridge. However, you can set up Jira automation or monitoring on the webhook status to alert you of failures.

**Q: Where can I see logs for debugging?**  
A: If you have access to the Forge development environment, use the Forge CLI command to view logs. End users typically don't have direct log access - rely on the "Last error" field in the UI.

### Integration Questions

**Q: Can I integrate AlertBridge with Grafana?**  
A: Yes, configure Grafana alert contact points to send webhooks to AlertBridge URL with the Bearer token.

**Q: Does AlertBridge work with PagerDuty or Opsgenie?**  
A: Not directly, but you can forward alerts from these services to AlertBridge if they support webhook forwarding in Alertmanager format.

**Q: Can I test AlertBridge without a real monitoring system?**  
A: Yes, use the curl test command provided on the AlertBridge configuration page.

---

## Summary

AlertBridge simplifies incident management by automatically creating and updating Jira issues from monitoring alerts. Key benefits:

✅ **Automatic issue creation** from alerts  
✅ **Smart deduplication** prevents duplicate issues  
✅ **Auto-resolution** when alerts clear  
✅ **Secure token authentication**  
✅ **Easy configuration** with no coding required  

### Quick Start Checklist

- [ ] Access AlertBridge configuration page in Jira
- [ ] Generate webhook token and copy it securely
- [ ] Select Jira project and issue type
- [ ] (Optional) Configure ignore label keys
- [ ] Save configuration
- [ ] Copy webhook URL
- [ ] Configure your monitoring system to send webhooks
- [ ] Test with curl command
- [ ] Verify issue creation in Jira
- [ ] Monitor webhook status for errors

### Getting Help

If you encounter issues not covered in this guide:

1. Check the "Webhook Status" section for error details
2. Review the Troubleshooting section above
3. Test with the provided curl command
4. Verify your Jira project and workflow configuration
5. Contact your Jira administrator or Atlassian Support

---

**Document Version:** 1.0  
**AlertBridge Version:** 1.1.31  
**Last Updated:** February 25, 2026

For technical or developer documentation, see the README.md file in the AlertBridge repository.

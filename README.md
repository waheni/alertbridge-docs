# 🚨 AlertBridge — Alerts → Jira (Forge App)

Turn Alertmanager-compatible webhook alerts into **deduplicated Jira issues** with automatic **resolve** and **reopen**.

---

## ✨ What AlertBridge does

When an alert arrives:

- 🔥 **Firing alert** → creates a Jira issue (first time only)
- 🔁 **Repeated firing** → adds a comment to the same issue (no spam)
- ✅ **Resolved** → adds a “resolved” comment and transitions the issue (if configured)
- ♻️ **Fires again after resolved** → reopens and comments

Designed to be **simple**, **secure**, and **production-safe**.

---

## 🔐 Security model (Token)

AlertBridge protects your webhook endpoint using a **Bearer token**.

- Token stored securely (Forge secret)
- UI shows masked token
- Regenerate anytime
- Treat like an API key

---

## 🛠️ Configuration

1. Go to **Jira settings → Apps → AlertBridge**
2. Select:
   - ✅ Project
   - ✅ Issue Type
3. Click **Save**
4. Click **Regenerate Token**
5. Copy **Webhook URL**

---

## 🚀 Quick Start (Alertmanager example)

```yaml
receivers:
  - name: alertbridge-jira
    webhook_configs:
      - url: <WEBHOOK_URL>
        http_config:
          authorization:
            type: Bearer
            credentials: <YOUR_TOKEN>

route:
  receiver: alertbridge-jira
```

---

## 🧪 Testing with curl

### 🔥 Firing alert

```bash
curl -i -X POST "<WEBHOOK_URL>" \
  -H "Authorization: Bearer <YOUR_TOKEN>" \
  -H "Content-Type: application/json" \
  --data-binary '{
    "status":"firing",
    "alerts":[{
      "status":"firing",
      "labels":{
        "alertname":"TestAlert",
        "severity":"critical",
        "service":"api",
        "env":"prod"
      },
      "annotations":{
        "summary":"Test alert",
        "description":"Testing AlertBridge"
      }
    }]
  }'
```

Expected:
- HTTP 200
- Jira issue created

---

### 🔁 Repeated firing → comment only

Send same payload again → existing issue updated.

---

### ✅ Resolve alert

```bash
curl -i -X POST "<WEBHOOK_URL>" \
  -H "Authorization: Bearer <YOUR_TOKEN>" \
  -H "Content-Type: application/json" \
  --data-binary '{
    "status":"resolved",
    "alerts":[{
      "status":"resolved",
      "labels":{
        "alertname":"TestAlert",
        "severity":"critical",
        "service":"api",
        "env":"prod"
      },
      "annotations":{
        "summary":"Resolved",
        "description":"Should resolve Jira issue"
      }
    }]
  }'
```

Expected:
- Comment added
- Issue transitioned if possible

---

### ♻️ Fire again → reopen

Send firing again → issue reopened.

---

## 🧠 Deduplication logic

Fingerprint built from alert **labels**.

Avoid duplicates by ignoring volatile labels like:
`instance,pod,container,node`

---

## ❤️ Health / Debugging

Health section shows:

- Last webhook received
- Last error
- Created / Updated counters

---

## 🧯 Troubleshooting

**Unauthorized**
→ Check `Authorization: Bearer <TOKEN>`

**No issues created**
→ Save Jira Target config

**Too many issues**
→ Ignore changing labels

**No transitions**
→ Workflow mismatch (comments still work)

---

## ❓ FAQ

**Need Prometheus?**
No. Any sender can POST compatible payload.

**Token hidden?**
Security best practice.

**Only curl testing?**
Fully supported.

---

## 📣 Support

support@yourdomain.com

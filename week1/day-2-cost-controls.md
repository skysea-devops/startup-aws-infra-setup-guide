# Day 2 — Cost Controls (Budgets)

## Goal
Avoid unexpected AWS bills.

---

## 1. Create Budget

### Console
Billing → Budgets → Create

### Setup
- Monthly budget (example: $500)

### Alerts
- 50% → Email
- 80% → Email + SNS
- 100% → Critical alert

---

## Recommendation

Use console for setup (CLI is error-prone for beginners).

---

## Verification

- Budget visible in console
- Alerts configured

----

#  Cost Monitoring (Advanced)

## 1. Cost Anomaly Detection

### Console
Billing → Cost Anomaly Detection

### Setup
- Monitor: AWS services
- Threshold: $10+

---

## 2. CloudWatch Billing Alarm

### IMPORTANT
Billing metrics only in us-east-1

### Enable first
Billing → Preferences → Enable Billing Alerts

---

### CLI

```bash
aws sns create-topic --name billing-alerts --region us-east-1
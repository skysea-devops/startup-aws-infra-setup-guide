

# Day 3-4 — CloudTrail Setup

## Goal
Log every API call.

---

## 1. Create S3 Log Bucket


---

# CloudTrail Advanced

## 1. Create Trail

### Console
CloudTrail → Create Trail

---

## Settings

- Multi-region → ENABLED
- Log validation → ENABLED
- Encryption → ENABLED

---

## Organization Trail

Only if Organizations is enabled.

---

## Warning

Data events (S3/Lambda):
→ Can increase cost significantly

---

## Verification

```bash
aws cloudtrail get-trail-status --name org-trail
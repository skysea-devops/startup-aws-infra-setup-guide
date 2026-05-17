
# Day 5 — AWS Organizations & Multi-Account Structure

## Goal
Prepare the AWS organization for multi-account infrastructure management.

---

## 1. Verify AWS Organizations is Enabled

Console

```
AWS Console → AWS Organizations
```

Verify that:

AWS Organizations is enabled
The current account is the management account

---

## 2. Create OUs

Console

```
AWS Organizations → Root
→ Children
→ Actions
→ Organizational unit
→ Create new
```

Recommended structure:

Root
└── Workloads
    ├── etg-dev
    └── etg-prod

---

## 3. Create AWS Accounts

Create the following accounts:

Account	Purpose
etg-dev	Development workloads
etg-prod	Production workloads

---

## 4. Assign Accounts to the Workloads OU

Move:

etg-dev
etg-prod

under:
```
Workloads OU
```
--- 

## 5. Configure IAM Identity Center Access

Assign appropriate Permission Sets for:

etg-dev
etg-prod

Example:

AdministratorAccess
PowerUserAccess
ReadOnlyAccess

---

## 6. Verify Cross-Account Access

Test:

IAM Identity Center login
Account switching
Access to dev/prod accounts


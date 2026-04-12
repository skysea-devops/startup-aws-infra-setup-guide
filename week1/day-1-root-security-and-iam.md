# Day 1 — Root Account Hardening & MFA

## Goal
Secure the most powerful credential in your AWS account.

---

## 1. Enable MFA on Root Account

### Console Path
Account → Security Credentials → MFA → Assign MFA

### Steps
1. Login with root account
2. Go to Security Credentials
3. Assign MFA device
4. Use:
   - Authenticator app

---

## Verification

### CLI (optional)
```bash
aws iam get-account-summary


----

# IAM Identity Center (SSO)

## Goal
Set up secure access.

---

## 1. Enable IAM Identity Center

### Console
Search → IAM Identity Center → Enable

### Important
Choose region carefully (cannot easily change later).

Recommended:
- eu-west-1
- us-east-1

---

## 2. Create Permission Sets

| Name      | Policy              | Use        |
|-----------|---------------------|------------|
| Admin     | AdministratorAccess | Infra team |
| PowerUser | PowerUserAccess     | Developers |
| ReadOnly  | ReadOnlyAccess      | Observers  |

---

## 3. Create User

IAM Identity Center → Users → Add user

---

## 4. Assign Access

IAM Identity Center → AWS Accounts → Assign

---

## 5. Enforce MFA

Settings → Authentication → Require MFA

---

## Verification

- Login via Access Portal
- MFA prompt appears
- Account access works

---

## Rule

> Never use IAM users. Always use SSO.
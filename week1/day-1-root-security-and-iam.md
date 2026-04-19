# Day 1 — Root Security & IAM Identity Center

## Goal
Set up secure access for your AWS account.

---

## 1. Enable MFA on Root Account

### Console
Account → Security Credentials → MFA → Assign MFA

### Steps
1. Log in with root account
2. Go to **Security Credentials**
3. Assign MFA device
4. Choose:
   - Authenticator app (recommended)
5. Scan QR code
6. Enter two consecutive codes

### Notes
- Never use root account for daily operations
- Use it only for:
  - Billing
  - Account-level changes
  - Emergency recovery
- **Do not create root access keys**

---

## 2. Enable IAM Identity Center

### Console
Search → IAM Identity Center → Enable

### Steps
1. Choose:
   - **Organization instance (recommended)**
2. Select region:
   - eu-west-1
   - us-east-1
3. Encryption:
   - Use AWS owned key

### Notes
- Region cannot be easily changed later
- This enables SSO-based access

---

## 3. Create Permission Sets

### Console
IAM Identity Center → Permission sets → Create permission set

### Recommended

| Name        | Policy                | Use          |
|------------ |---------------------- |--------------|
| Admin       | AdministratorAccess   | Infra / Owner |
| PowerUser   | PowerUserAccess       | DevOps       |

### Notes
- Permission sets = role templates
- Applied per account
- Translated into IAM roles behind the scenes

---

## 4. Create User

### Console
IAM Identity Center → Users → Add user

### Steps
1. Enter:
   - Username
   - Email
   - First & Last name
2. Send invitation

### Notes
- Users do NOT get direct permissions
- Access is defined via permission sets

---

## 5. Assign Access

### Console
IAM Identity Center → AWS Accounts → Assign users or groups

### Steps
1. Select AWS account
2. Select user
3. Select permission set
4. Confirm

### Result
User → Account → Role mapping is created

---

## 6. Enforce MFA

### Console
IAM Identity Center → Settings → Authentication → Require MFA

### Notes
- Applies to all users
- Required for production setups

---

## 7. Access Portal Login

Users will log in via a link like this: https://xxxx.awsapps.com/start

### Notes
- This is your new AWS login entry point
- Do NOT use IAM user login


# Day 5 — Terraform Remote State Bootstrap

Building Terraform’s remote state infrastructure.

---

## Prerequisites

- AWS CLI installed and configured with IAM Identity Center (SSO)
- Terraform >= 1.10
- Admin access on management account

Verify Terraform version:

```bash
terraform -version
```

---

## 1. Configure AWS CLI with SSO

```bash
aws configure sso
```

You will be prompted for SSO start URL, region, account, and role.

Login:

```bash
aws sso login --profile management
```

Verify:

```bash
aws sts get-caller-identity --profile management
```

Set the profile for Terraform:

```bash
export AWS_PROFILE=management
```

---

## 2. Create Project Structure

```bash
mkdir -p terraform-state-bootstrap
cd terraform-state-bootstrap
```

Structure:

```
terraform-state-bootstrap/
├── main.tf
├── variables.tf
└── outputs.tf
```

---

## 3. Create variables.tf

```hcl
variable "project_name" {
  description = "Project name used in resource naming"
  type        = string
  default     = "startup"
}

variable "aws_region" {
  description = "AWS region for the S3 bucket"
  type        = string
  default     = "eu-west-1"
}

variable "tags" {
  description = "Tags applied to all resources"
  type        = map(string)
  default = {
    Project     = "infrastructure"
    ManagedBy   = "terraform"
    Environment = "management"
  }
}
```

---

## 4. Create main.tf

S3 bucket with versioning, KMS encryption, public access block, SSL-only policy, and delete protection.

```hcl
terraform {
  required_version = ">= 1.10"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = var.aws_region
}

data "aws_caller_identity" "current" {}

# --- S3 Bucket ---

resource "aws_s3_bucket" "terraform_state" {
  bucket = "${var.project_name}-terraform-state-${data.aws_caller_identity.current.account_id}"

  lifecycle {
    prevent_destroy = true
  }

  tags = var.tags
}

# --- Versioning ---

resource "aws_s3_bucket_versioning" "terraform_state" {
  bucket = aws_s3_bucket.terraform_state.id

  versioning_configuration {
    status = "Enabled"
  }
}

# --- KMS Encryption ---

resource "aws_s3_bucket_server_side_encryption_configuration" "terraform_state" {
  bucket = aws_s3_bucket.terraform_state.id

  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "aws:kms"
    }
    bucket_key_enabled = true
  }
}

# --- Public Access Block ---

resource "aws_s3_bucket_public_access_block" "terraform_state" {
  bucket = aws_s3_bucket.terraform_state.id

  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

# --- SSL-Only Bucket Policy ---

resource "aws_s3_bucket_policy" "enforce_ssl" {
  bucket = aws_s3_bucket.terraform_state.id

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Sid       = "EnforceSSLOnly"
        Effect    = "Deny"
        Principal = "*"
        Action    = "s3:*"
        Resource = [
          aws_s3_bucket.terraform_state.arn,
          "${aws_s3_bucket.terraform_state.arn}/*"
        ]
        Condition = {
          Bool = {
            "aws:SecureTransport" = "false"
          }
        }
      }
    ]
  })
}
```

---

## 5. Create outputs.tf

```hcl
output "state_bucket_name" {
  description = "Terraform state bucket name"
  value       = aws_s3_bucket.terraform_state.id
}

output "state_bucket_arn" {
  description = "Terraform state bucket ARN"
  value       = aws_s3_bucket.terraform_state.arn
}

output "state_bucket_region" {
  description = "Bucket region"
  value       = var.aws_region
}
```

---

## 6. Apply

```bash
terraform init
terraform plan
terraform apply
```

Save the bucket name and ARN from the output.

> **Note:** This bootstrap project stores its state locally at first. This is expected — the bucket doesn't exist yet when Terraform runs for the first time.

---

## 7. Migrate State to Its Own Bucket

Add the backend block to `main.tf`:

```hcl
terraform {
  required_version = ">= 1.10"

  backend "s3" {
    bucket       = "startup-terraform-state-123456789012"  # your bucket name
    key          = "bootstrap/terraform.tfstate"
    region       = "eu-west-1"
    use_lockfile = true
    encrypt      = true
  }

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}
```

Migrate:

```bash
terraform init -migrate-state
```

Type `yes` when prompted.

---

## 8. Verify

```bash
# Bucket exists
aws s3 ls | grep terraform-state

# Versioning enabled
aws s3api get-bucket-versioning \
  --bucket startup-terraform-state-123456789012

# Public access blocked
aws s3api get-public-access-block \
  --bucket startup-terraform-state-123456789012

# Encryption enabled
aws s3api get-bucket-encryption \
  --bucket startup-terraform-state-123456789012
```

Expected results:

- Versioning → `"Status": "Enabled"`
- Public access → all four fields `true`
- Encryption → `"SSEAlgorithm": "aws:kms"`

---

## 9. Using This Bucket in Other Projects

Example backend config for a dev networking project:

```hcl
terraform {
  required_version = ">= 1.10"

  backend "s3" {
    bucket       = "startup-terraform-state-123456789012"
    key          = "dev/networking/terraform.tfstate"
    region       = "eu-west-1"
    use_lockfile = true
    encrypt      = true
  }
}
```

Recommended key structure:

```
startup-terraform-state-123456789012/
├── bootstrap/terraform.tfstate
├── dev/
│   ├── networking/terraform.tfstate
│   ├── application/terraform.tfstate
│   └── database/terraform.tfstate
└── prod/
    ├── networking/terraform.tfstate
    ├── application/terraform.tfstate
    └── database/terraform.tfstate
```

---

## Why No DynamoDB?

Terraform 1.10+ supports native S3 locking with `use_lockfile = true`. No need for a DynamoDB lock table in new setups.

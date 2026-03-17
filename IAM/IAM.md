# 🔐 AWS IAM — Complete Hands-On Guide

> **Author:** Teja Balamanasa | Senior Cloud DevSecOps Engineer | [Teja Technologies](https://github.com/teja-technologies)
>
> ⭐ **Star this repo** if it helped you! Share with your team and the AWS community.
>
> 🏷️ **Tags:** `aws` `iam` `cloud-security` `devops` `devsecops` `aws-community-builder` `cloud-computing` `hands-on` `beginners` `interview-prep`

---

## 📋 Table of Contents

| # | Topic | Description |
|---|-------|-------------|
| 1 | [What is AWS IAM?](#1-what-is-aws-iam) | Foundation, concepts, why it matters |
| 2 | [IAM Users](#2-iam-users) | Creating users, credentials, best practices |
| 3 | [IAM Groups](#3-iam-groups) | Group management, team-based access |
| 4 | [IAM Policies](#4-iam-policies) | JSON policies, types, deny rules |
| 5 | [IAM Roles](#5-iam-roles) | Temporary credentials, STS, use cases |
| 6 | [MFA & Password Policy](#6-mfa--password-policy) | Security hardening |
| 7 | [IAM Conditions](#7-iam-conditions) | Context-aware access control |
| 8 | [Tags & ABAC](#8-tags--abac) | Attribute-based access at scale |
| 9 | [Hands-On Labs](#9-hands-on-labs) | Step-by-step CLI and Console exercises |
| 10 | [Real-World Scenarios](#10-real-world-scenarios) | Banking, startup, enterprise patterns |
| 11 | [Interview Q&A](#11-interview-qa) | Top 20 IAM interview questions |
| 12 | [Security Best Practices](#12-security-best-practices) | Do's and Don'ts |
| 13 | [Quick Reference Cheat Sheet](#13-quick-reference-cheat-sheet) | Summary cards |

---

## 1. What is AWS IAM?

AWS **Identity and Access Management (IAM)** is a **free, global AWS service** that lets you control who is authenticated (signed in) and authorized (has permissions) to use AWS resources.

### 🎯 Three Pillars of IAM

```
┌─────────────────────────────────────────────────────────────────────┐
│                        AWS IAM                                      │
│                                                                     │
│   🔑 AUTHENTICATION      🔒 AUTHORIZATION       📋 AUDIT           │
│   ─────────────────      ──────────────────      ─────────         │
│   WHO are you?           WHAT can you do?        WHAT did you do?  │
│                                                                     │
│   Verify identity        Control actions         CloudTrail logs   │
│   via credentials        on resources            all activity      │
└─────────────────────────────────────────────────────────────────────┘
```

### 🏢 Layman Analogy

Think of IAM like a **corporate office building**:

```
🏢 Office Building = AWS Account
🪪 ID Card         = IAM User
👥 Department      = IAM Group
📋 Access Rules    = IAM Policy
🎫 Visitor Badge   = IAM Role (temporary)
🔐 Door Lock       = Resource Permission
```

> **Example:** A janitor has keys to every floor but cannot access the CEO's safe.
> A developer has access to the dev environment but cannot modify production billing.

### 🌍 Why IAM Matters

| Without IAM | With IAM |
|-------------|----------|
| Everyone has full access | Fine-grained access control |
| No audit trail | Every action is logged |
| One breach = total loss | Blast radius is minimized |
| No compliance possible | PCI-DSS, SOC2, HIPAA ready |

### 🔑 Key Facts

- ✅ IAM is **FREE** — no additional charge
- ✅ IAM is **Global** — not region-specific
- ✅ Default behavior — **everything is DENIED** unless explicitly allowed
- ✅ Root account = **God mode** — protect it with extreme care
- ✅ Supports **identity federation** (Active Directory, Google, GitHub)

---

## 2. IAM Users

An **IAM User** is a permanent identity representing a **person** or **application** in your AWS account.

### 🗂️ Architecture Diagram

```
                        AWS Account
                            │
                    ┌───────┴────────┐
                    │   Root User    │  ← 🚨 Never use daily
                    └───────┬────────┘
                            │ creates
              ┌─────────────┼──────────────┐
              │             │              │
        ┌─────┴─────┐ ┌─────┴─────┐ ┌─────┴─────┐
        │  IAM User │ │  IAM User │ │  IAM User │
        │  ravi     │ │  priya    │ │  jenkins  │
        │  (human)  │ │  (human)  │ │  (app)    │
        └─────┬─────┘ └─────┬─────┘ └─────┬─────┘
              │             │              │
        Console         Console        Access Keys
        Password        Password       (CLI/API)
```

### 🔐 User Credential Types

| Credential | Use Case | When to Use |
|------------|----------|-------------|
| **Console Password** | Login to AWS Web Console | Human users |
| **Access Key ID + Secret** | CLI, SDK, API, CI/CD | Applications, automation |
| **SSH Key** | CodeCommit Git access | Developers using Git |
| **X.509 Certificate** | SOAP-based APIs | Legacy use cases |

> ⚠️ **A single user can have a maximum of 2 access keys at a time.**

### 💻 Hands-On: Create IAM User via CLI

```bash
# Step 1: Create the user
aws iam create-user --user-name ravi.kumar

# Step 2: Create a login profile (console password)
aws iam create-login-profile \
  --user-name ravi.kumar \
  --password "SecurePass@2024!" \
  --password-reset-required

# Step 3: Create access keys (for CLI/API use)
aws iam create-access-key --user-name ravi.kumar

# Step 4: List all users
aws iam list-users --output table

# Step 5: Get details about a specific user
aws iam get-user --user-name ravi.kumar
```

### 🖥️ Hands-On: Create IAM User via Console

```
1. Login to AWS Console → https://console.aws.amazon.com
2. Search "IAM" in the services search bar
3. Click "Users" in the left sidebar
4. Click "Create user" button (top right)
5. Enter User name: ravi.kumar
6. Check "Provide user access to the AWS Management Console"
7. Select "I want to create an IAM user"
8. Set custom password or auto-generate
9. Uncheck "Users must create a new password at next sign-in" (for lab)
10. Click "Next" → Set permissions → Click "Next" → "Create user"
11. Download .csv with credentials (ONLY shown once!)
```

### 🏦 Real-World Example (Banking App)

```yaml
Scenario: Capital One AWS Environment

Users:
  ravi.kumar@company.com:
    type: Human (Developer)
    access: Console login
    permissions: S3 read/write in DEV only
    mfa: Enabled (Google Authenticator)

  jenkins-cicd:
    type: Application (CI/CD Pipeline)
    access: Access Keys only
    permissions: S3 PutObject, ECR PushImage
    mfa: Not applicable

  priya.dba@company.com:
    type: Human (DBA)
    access: Console login
    permissions: RDS full access, no EC2
    mfa: Enabled (Hardware YubiKey)

  audit-readonly:
    type: Application (Audit Tool)
    access: Access Keys
    permissions: ReadOnlyAccess managed policy
    mfa: Not applicable
```

### ✅ IAM User Best Practices

```
✅ DO:
  ├── Create individual users (one per person, one per service)
  ├── Enable MFA for all human users
  ├── Rotate access keys every 90 days
  ├── Use strong passwords (14+ chars, mixed)
  └── Delete unused users immediately

❌ DON'T:
  ├── Never use root account for daily work
  ├── Never share credentials between team members
  ├── Never hardcode access keys in source code
  ├── Never store access keys in GitHub repositories
  └── Never leave inactive users with active keys
```

---

## 3. IAM Groups

An **IAM Group** is a collection of IAM users. Attach policies to the group — all members automatically inherit the permissions.

### 🗂️ Architecture Diagram

```
                        IAM Groups Structure
                               │
         ┌─────────────────────┼─────────────────────┐
         │                     │                     │
  ┌──────┴──────┐       ┌──────┴──────┐       ┌──────┴──────┐
  │  Developers │       │    DBAs     │       │  Auditors   │
  │    Group    │       │   Group     │       │   Group     │
  └──────┬──────┘       └──────┬──────┘       └──────┬──────┘
         │                     │                     │
  PowerUserAccess       RDSFullAccess          ReadOnlyAccess
  (AWS Managed)         (AWS Managed)          (AWS Managed)
         │                     │                     │
  ┌──────┴──────┐       ┌──────┴──────┐       ┌──────┴──────┐
  │ ravi.kumar  │       │ priya.dba   │       │ audit-user1 │
  │ sneha.dev   │       │ suresh.dba  │       │             │
  │ aman.dev    │       │             │       │             │
  └─────────────┘       └─────────────┘       └─────────────┘
```

### 💻 Hands-On: Create Groups and Add Users

```bash
# Step 1: Create a group
aws iam create-group --group-name Developers

# Step 2: Attach an AWS Managed Policy to the group
aws iam attach-group-policy \
  --group-name Developers \
  --policy-arn arn:aws:iam::aws:policy/PowerUserAccess

# Step 3: Add a user to the group
aws iam add-user-to-group \
  --group-name Developers \
  --user-name ravi.kumar

# Step 4: Add another user
aws iam add-user-to-group \
  --group-name Developers \
  --user-name sneha.dev

# Step 5: List users in a group
aws iam get-group --group-name Developers

# Step 6: List groups a user belongs to
aws iam list-groups-for-user --user-name ravi.kumar

# Step 7: Remove user from group (when they leave)
aws iam remove-user-from-group \
  --group-name Developers \
  --user-name ravi.kumar
```

### 📌 Critical Rules — Common Exam & Interview Questions

```
Rule 1: Groups can ONLY contain USERS — NOT other groups
        ✅ Group → Users
        ❌ Group → Groups (NOT allowed)

Rule 2: A user CAN belong to multiple groups
        ✅ ravi.kumar → Developers group
        ✅ ravi.kumar → SecurityTeam group
        Both sets of permissions are combined (union)

Rule 3: Groups do NOT have credentials
        Groups cannot be used to login
        Permissions flow from Group → User only

Rule 4: Max 300 groups per AWS account (soft limit)
Rule 5: A user can belong to max 10 groups
```

### 🏢 Real-World Team Onboarding Workflow

```bash
# New developer joins the team
echo "=== Onboarding: New Developer ==="

# 1. Create user
aws iam create-user --user-name john.doe

# 2. Set console password
aws iam create-login-profile \
  --user-name john.doe \
  --password "Welcome@2024!" \
  --password-reset-required

# 3. Add to developer group (gets all dev permissions instantly)
aws iam add-user-to-group \
  --group-name Developers \
  --user-name john.doe

echo "✅ john.doe is onboarded with Developer access"

# When developer transfers to DevOps team
echo "=== Team Transfer ==="
aws iam remove-user-from-group \
  --group-name Developers \
  --user-name john.doe

aws iam add-user-to-group \
  --group-name DevOps \
  --user-name john.doe

# When developer leaves the company
echo "=== Offboarding ==="
aws iam remove-user-from-group \
  --group-name DevOps \
  --user-name john.doe

aws iam delete-login-profile --user-name john.doe

# Deactivate access keys
aws iam list-access-keys --user-name john.doe
# Then delete each key:
aws iam delete-access-key \
  --user-name john.doe \
  --access-key-id AKIAIOSFODNN7EXAMPLE
```

---

## 4. IAM Policies

An **IAM Policy** is a JSON document that defines **what actions** are allowed or denied on **which resources** under **what conditions**.

### 🗂️ Policy Structure

```
┌─────────────────────────────────────────────────────┐
│                    IAM Policy                       │
│                                                     │
│  {                                                  │
│    "Version": "2012-10-17",        ← Always this   │
│    "Statement": [                  ← List of rules  │
│      {                                              │
│        "Sid": "AllowS3Read",       ← Optional ID   │
│        "Effect": "Allow",          ← Allow/Deny    │
│        "Action": ["s3:GetObject"], ← What to do    │
│        "Resource": "arn:...",      ← On what       │
│        "Condition": {...}          ← Optional      │
│      }                                              │
│    ]                                                │
│  }                                                  │
└─────────────────────────────────────────────────────┘
```

### 📝 Policy Examples

#### Example 1 — S3 Read-Only Access to Specific Bucket

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowS3ReadOnly",
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:ListBucket",
        "s3:GetBucketLocation"
      ],
      "Resource": [
        "arn:aws:s3:::my-company-bucket",
        "arn:aws:s3:::my-company-bucket/*"
      ]
    }
  ]
}
```

#### Example 2 — EC2 Full Access in a Specific Region

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "EC2FullAccessUSEast1",
      "Effect": "Allow",
      "Action": "ec2:*",
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "aws:RequestedRegion": "us-east-1"
        }
      }
    }
  ]
}
```

#### Example 3 — Deny Delete on Production Resources

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyDeleteProduction",
      "Effect": "Deny",
      "Action": [
        "s3:DeleteObject",
        "s3:DeleteBucket",
        "ec2:TerminateInstances",
        "rds:DeleteDBInstance"
      ],
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "aws:ResourceTag/Environment": "Production"
        }
      }
    }
  ]
}
```

#### Example 4 — Developer Sandbox Policy (Self-Service)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowEC2Sandbox",
      "Effect": "Allow",
      "Action": [
        "ec2:RunInstances",
        "ec2:StopInstances",
        "ec2:StartInstances",
        "ec2:DescribeInstances"
      ],
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "ec2:InstanceType": ["t2.micro", "t3.micro"]
        }
      }
    },
    {
      "Sid": "DenyLargeInstances",
      "Effect": "Deny",
      "Action": "ec2:RunInstances",
      "Resource": "arn:aws:ec2:*:*:instance/*",
      "Condition": {
        "ForAnyValue:StringNotLike": {
          "ec2:InstanceType": ["t2.micro", "t3.micro"]
        }
      }
    }
  ]
}
```

### 🏷️ Policy Types Comparison

```
┌─────────────────┬──────────────────┬──────────────────┬──────────────────┐
│                 │  AWS Managed     │ Customer Managed │ Inline Policy    │
├─────────────────┼──────────────────┼──────────────────┼──────────────────┤
│ Who creates?    │ AWS              │ You              │ You              │
│ Who maintains?  │ AWS (auto-upd)   │ You              │ You              │
│ Reusable?       │ Yes              │ Yes              │ No (1-to-1)      │
│ Versioning?     │ No               │ Yes (up to 5)    │ No               │
│ When to use?    │ Common use cases │ Custom needs     │ Tight coupling   │
│ Examples        │ ReadOnlyAccess   │ AppSpecificRole  │ Emergency access │
│                 │ PowerUserAccess  │ DataScienceAccess│                  │
└─────────────────┴──────────────────┴──────────────────┴──────────────────┘
```

### ⚡ Policy Evaluation Logic

```
┌─────────────────────────────────────────────────────────────────────┐
│              AWS Policy Evaluation Flow                             │
│                                                                     │
│  Request comes in                                                   │
│        │                                                            │
│        ▼                                                            │
│  ┌─────────────┐  YES                                               │
│  │ Explicit    ├──────────────────────────────► DENY ❌            │
│  │ DENY exists?│                                                    │
│  └──────┬──────┘                                                    │
│         │ NO                                                        │
│         ▼                                                           │
│  ┌─────────────┐  NO                                                │
│  │ Explicit    ├──────────────────────────────► DENY ❌            │
│  │ ALLOW exists│                                                    │
│  └──────┬──────┘                                                    │
│         │ YES                                                       │
│         ▼                                                           │
│       ALLOW ✅                                                      │
│                                                                     │
│  🔑 KEY RULE: Explicit DENY always wins over ALLOW                  │
└─────────────────────────────────────────────────────────────────────┘
```

### 💻 Hands-On: Create and Attach Custom Policy

```bash
# Step 1: Save policy to a file
cat > s3-read-policy.json << 'EOF'
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowS3ReadOnly",
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::my-company-bucket",
        "arn:aws:s3:::my-company-bucket/*"
      ]
    }
  ]
}
EOF

# Step 2: Create the customer managed policy
aws iam create-policy \
  --policy-name S3ReadOnlyPolicy \
  --policy-document file://s3-read-policy.json \
  --description "Allows read-only access to my-company-bucket"

# Step 3: Note the Policy ARN from the output, then attach it
aws iam attach-user-policy \
  --user-name ravi.kumar \
  --policy-arn arn:aws:iam::123456789012:policy/S3ReadOnlyPolicy

# Attach to a group instead (recommended)
aws iam attach-group-policy \
  --group-name Developers \
  --policy-arn arn:aws:iam::123456789012:policy/S3ReadOnlyPolicy

# Step 4: List policies attached to a user
aws iam list-attached-user-policies --user-name ravi.kumar

# Step 5: Simulate policy (check if permission is allowed)
aws iam simulate-principal-policy \
  --policy-source-arn arn:aws:iam::123456789012:user/ravi.kumar \
  --action-names s3:GetObject \
  --resource-arns arn:aws:s3:::my-company-bucket/myfile.txt
```

---

## 5. IAM Roles

An **IAM Role** is a **temporary identity** — no username or password. Services, applications, or users **assume** a role to get short-lived credentials via **AWS STS (Security Token Service)**.

### 🗂️ Architecture Diagram

```
                        IAM Roles — How It Works
                                    │
                    ┌───────────────┴───────────────┐
                    │         Who can assume?        │
                    └───────────────┬───────────────┘
         ┌──────────────────────────┼──────────────────────────┐
         │                          │                          │
   ┌─────┴─────┐             ┌──────┴──────┐           ┌──────┴──────┐
   │ AWS       │             │  IAM User   │           │  External   │
   │ Services  │             │  /Account   │           │  Identity   │
   │ (EC2,     │             │ (Cross-acct)│           │  (SAML/OIDC)│
   │ Lambda)   │             │             │           │             │
   └─────┬─────┘             └──────┬──────┘           └──────┬──────┘
         │                          │                          │
         └──────────────────────────┼──────────────────────────┘
                                    │
                              Assume Role API
                                    │
                                    ▼
                          ┌─────────────────┐
                          │  AWS STS        │
                          │  (Token Service)│
                          └────────┬────────┘
                                   │ Issues
                                   ▼
                    ┌──────────────────────────────┐
                    │  Temporary Credentials       │
                    │  ├── Access Key ID           │
                    │  ├── Secret Access Key       │
                    │  ├── Session Token           │
                    │  └── Expiration (15m-12hr)   │
                    └──────────────────────────────┘
```

### 🎯 Role Use Cases

#### Use Case 1 — EC2 Instance accessing S3 (Most Common)

```
Traditional (BAD):                   Using Role (GOOD):
                                     
  EC2 Instance                         EC2 Instance
      │                                    │
  Access Key                          IAM Role attached
  hardcoded                           at launch
  in config                               │
      │                               Automatic temp
      ▼                               credentials
    S3 Bucket                               │
  (keys can leak                           ▼
   to GitHub!)                          S3 Bucket ✅
```

```bash
# Create trust policy (who can assume this role)
cat > ec2-trust-policy.json << 'EOF'
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "ec2.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
EOF

# Create the role
aws iam create-role \
  --role-name EC2-S3-Upload-Role \
  --assume-role-policy-document file://ec2-trust-policy.json \
  --description "Allows EC2 to upload files to S3"

# Attach permissions to the role
aws iam attach-role-policy \
  --role-name EC2-S3-Upload-Role \
  --policy-arn arn:aws:iam::aws:policy/AmazonS3FullAccess

# Create instance profile (needed to attach role to EC2)
aws iam create-instance-profile \
  --instance-profile-name EC2-S3-Profile

aws iam add-role-to-instance-profile \
  --instance-profile-name EC2-S3-Profile \
  --role-name EC2-S3-Upload-Role

# Launch EC2 with the role
aws ec2 run-instances \
  --image-id ami-0abcdef1234567890 \
  --instance-type t2.micro \
  --iam-instance-profile Name=EC2-S3-Profile
```

#### Use Case 2 — Cross-Account Access

```
┌─────────────────────┐         ┌─────────────────────┐
│  Account A (DEV)    │         │  Account B (PROD)   │
│  ID: 111111111111   │         │  ID: 222222222222   │
│                     │         │                     │
│  IAM User: dev-user │──────── ► IAM Role: ProdAudit │
│                     │ assumes │  Permissions:       │
│                     │         │  ReadOnlyAccess     │
└─────────────────────┘         └─────────────────────┘
```

```bash
# In Account B (PROD) — Create role that Account A can assume
cat > cross-account-trust.json << 'EOF'
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::111111111111:user/dev-user"
      },
      "Action": "sts:AssumeRole",
      "Condition": {
        "Bool": {
          "aws:MultiFactorAuthPresent": "true"
        }
      }
    }
  ]
}
EOF

aws iam create-role \
  --role-name ProdAuditRole \
  --assume-role-policy-document file://cross-account-trust.json

aws iam attach-role-policy \
  --role-name ProdAuditRole \
  --policy-arn arn:aws:iam::aws:policy/ReadOnlyAccess

# In Account A (DEV) — Assume the role
aws sts assume-role \
  --role-arn arn:aws:iam::222222222222:role/ProdAuditRole \
  --role-session-name audit-session-2024 \
  --duration-seconds 3600

# Set environment variables with the returned credentials
export AWS_ACCESS_KEY_ID="ASIAX..."
export AWS_SECRET_ACCESS_KEY="..."
export AWS_SESSION_TOKEN="..."

# Now you're operating as the ProdAuditRole in Account B
aws s3 ls  # Lists S3 in PROD account
```

#### Use Case 3 — Lambda Function Role

```bash
# Trust policy for Lambda
cat > lambda-trust-policy.json << 'EOF'
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "lambda.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
EOF

# Create execution role
aws iam create-role \
  --role-name LambdaDynamoDBRole \
  --assume-role-policy-document file://lambda-trust-policy.json

# Attach permissions
aws iam attach-role-policy \
  --role-name LambdaDynamoDBRole \
  --policy-arn arn:aws:iam::aws:policy/AmazonDynamoDBReadOnlyAccess

# Also attach basic Lambda execution (for CloudWatch logs)
aws iam attach-role-policy \
  --role-name LambdaDynamoDBRole \
  --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
```

### 🎫 Role vs User — Key Differences

```
┌──────────────────────────┬──────────────────────────┐
│       IAM USER           │       IAM ROLE           │
├──────────────────────────┼──────────────────────────┤
│ Permanent identity       │ Temporary identity       │
│ Has password/keys        │ No password/keys         │
│ For humans & apps        │ For services & users     │
│ Long-term credentials    │ Short-term (15m-12h)     │
│ Direct permissions       │ Assumed permissions      │
│ Specific to one account  │ Can span accounts        │
└──────────────────────────┴──────────────────────────┘
```

---

## 6. MFA & Password Policy

### 🔐 Multi-Factor Authentication (MFA)

MFA adds a **second layer** of security. Even if a password is stolen, the account remains protected.

```
Login Without MFA:          Login With MFA:
                            
  Password ──► Access ✅     Password + TOTP ──► Access ✅
  Stolen pwd ──► Hacked ❌   Stolen pwd alone ──► DENIED ✅
```

### MFA Device Types

| Type | Device | Security Level | Recommended For |
|------|--------|---------------|-----------------|
| **Virtual MFA** | Google Authenticator, Authy | Medium | All users |
| **Hardware MFA** | YubiKey, Gemalto | High | Root, Admins |
| **FIDO2/WebAuthn** | USB Security Key | Very High | Privileged access |
| **SMS MFA** | Phone number | Low | Avoid if possible |

### 💻 Enable MFA via CLI

```bash
# Step 1: Create a virtual MFA device
aws iam create-virtual-mfa-device \
  --virtual-mfa-device-name ravi-mfa \
  --outfile /tmp/ravi-qr.png \
  --bootstrap-method QRCodePNG

# Scan the QR code with Google Authenticator app
# Get two consecutive TOTP codes: CODE1 and CODE2

# Step 2: Enable MFA for the user
aws iam enable-mfa-device \
  --user-name ravi.kumar \
  --serial-number arn:aws:iam::123456789012:mfa/ravi-mfa \
  --authentication-code1 123456 \
  --authentication-code2 789012

# Step 3: Verify MFA is enabled
aws iam list-mfa-devices --user-name ravi.kumar
```

### 🔑 Password Policy Setup

```bash
# Set account-level password policy
aws iam update-account-password-policy \
  --minimum-password-length 14 \
  --require-symbols \
  --require-numbers \
  --require-uppercase-characters \
  --require-lowercase-characters \
  --allow-users-to-change-password \
  --max-password-age 60 \
  --password-reuse-prevention 5

# View current password policy
aws iam get-account-password-policy
```

### 🏢 Enterprise Password Policy (Recommended)

```yaml
PasswordPolicy:
  MinimumLength: 14
  RequireUppercase: true
  RequireLowercase: true
  RequireNumbers: true
  RequireSymbols: true
  MaxPasswordAge: 60        # Days before expiry
  PasswordReusePrevention: 5  # Can't reuse last 5
  AllowUsersToChange: true
  HardExpiry: false         # Don't lock out on expiry
```

### ⚠️ Root Account Security Checklist

```
Root Account Security — Do This First!

[ ] Enable MFA on root account (virtual or hardware)
[ ] Delete root access keys (if they exist)
[ ] Do NOT create programmatic access for root
[ ] Store root credentials in a secure vault (e.g., 1Password, HashiCorp Vault)
[ ] Enable billing alerts
[ ] Use root ONLY for these specific tasks:
    - Close the AWS account
    - Change the account email/name
    - Restore IAM admin access if locked out
    - Enable IAM access to Billing console
    - Register as a seller in the Reserved Instance Marketplace
```

```bash
# Check if root has access keys (security audit)
aws iam get-account-summary | grep -i root

# Generate credential report to audit all users
aws iam generate-credential-report

# Download and view the report
aws iam get-credential-report \
  --query 'Content' \
  --output text | base64 -d > credential-report.csv

# View it
cat credential-report.csv
```

---

## 7. IAM Conditions

**Conditions** make policies context-aware — restrict access based on IP address, time, MFA status, region, tags, and more.

### 🗂️ Condition Structure

```json
"Condition": {
  "ConditionOperator": {
    "ConditionKey": "ConditionValue"
  }
}
```

### 🔧 Common Condition Operators

| Operator | Use Case |
|----------|----------|
| `StringEquals` | Exact string match |
| `StringLike` | Pattern match with wildcards |
| `IpAddress` | IP range match |
| `Bool` | True/false check |
| `DateGreaterThan` | Time-based access |
| `ArnLike` | ARN pattern match |
| `Null` | Check if key exists |

### 📝 Condition Examples

#### 1 — Allow S3 Access Only from Office IP

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowS3FromOfficeOnly",
      "Effect": "Allow",
      "Action": "s3:*",
      "Resource": "*",
      "Condition": {
        "IpAddress": {
          "aws:SourceIp": [
            "203.0.113.0/24",
            "198.51.100.0/24"
          ]
        }
      }
    }
  ]
}
```

#### 2 — Require MFA for Sensitive Operations

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyWithoutMFA",
      "Effect": "Deny",
      "Action": [
        "ec2:StopInstances",
        "ec2:TerminateInstances",
        "rds:DeleteDBInstance",
        "s3:DeleteBucket"
      ],
      "Resource": "*",
      "Condition": {
        "BoolIfExists": {
          "aws:MultiFactorAuthPresent": "false"
        }
      }
    }
  ]
}
```

#### 3 — Allow Only During Business Hours

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "BusinessHoursOnly",
      "Effect": "Allow",
      "Action": "ec2:*",
      "Resource": "*",
      "Condition": {
        "DateGreaterThan": {
          "aws:CurrentTime": "2024-01-01T09:00:00Z"
        },
        "DateLessThan": {
          "aws:CurrentTime": "2024-12-31T18:00:00Z"
        }
      }
    }
  ]
}
```

#### 4 — Force HTTPS Only for S3

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyHTTP",
      "Effect": "Deny",
      "Action": "s3:*",
      "Resource": [
        "arn:aws:s3:::my-secure-bucket",
        "arn:aws:s3:::my-secure-bucket/*"
      ],
      "Condition": {
        "Bool": {
          "aws:SecureTransport": "false"
        }
      }
    }
  ]
}
```

#### 5 — Restrict to Specific Region

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyOutsideUSEast1",
      "Effect": "Deny",
      "NotAction": [
        "iam:*",
        "sts:*",
        "cloudfront:*",
        "route53:*"
      ],
      "Resource": "*",
      "Condition": {
        "StringNotEquals": {
          "aws:RequestedRegion": "us-east-1"
        }
      }
    }
  ]
}
```

### 🔑 Important Condition Keys Reference

```
Global Condition Keys:
├── aws:SourceIp           — Originating IP address
├── aws:SourceVpc          — Request from specific VPC
├── aws:RequestedRegion    — Target region for the request
├── aws:CurrentTime        — Current date/time
├── aws:MultiFactorAuthPresent — MFA was used
├── aws:SecureTransport    — HTTPS used (true/false)
├── aws:PrincipalTag/{key} — Tag on the calling user/role
├── aws:ResourceTag/{key}  — Tag on the resource
└── aws:CalledVia          — Service making the request

Service-Specific Keys:
├── s3:prefix              — S3 object key prefix
├── ec2:InstanceType       — EC2 instance type
├── ec2:Region             — EC2 region
├── rds:DatabaseClass      — RDS instance class
└── kms:CallerAccount      — KMS caller account ID
```

---

## 8. Tags & ABAC

### 🏷️ What are Tags?

Tags are **key-value pairs** that label AWS resources and IAM identities. They enable:
- Cost allocation and billing
- Resource organization
- Access control (ABAC)
- Automation triggers

```
Key         Value
──────────  ──────────
Department  Engineering
Team        DevOps
Environment Production
Project     Phoenix
CostCenter  CC-1042
Owner       ravi.kumar
```

### 🔐 ABAC — Attribute-Based Access Control

ABAC uses **tags** to make policies dynamic and scalable.

```
Traditional RBAC (Role-Based):          ABAC (Attribute-Based):
                                        
100 teams → 100 policies                1 policy → checks tags
Policy explosion! 😱                    Add team? Just tag! 😊
                                        
Hard to scale                           Scales infinitely
Manual policy updates                   Auto-applies via tags
```

### 🗂️ ABAC Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                    ABAC in Action                                   │
│                                                                     │
│  IAM User: ravi.kumar           EC2 Instance: i-abc123             │
│  Tags:                          Tags:                               │
│    Department = Engineering       Department = Engineering          │
│    Team       = DevOps            Team       = DevOps               │
│                                                                     │
│  ABAC Policy:                                                       │
│  "Allow EC2 actions IF                                              │
│   user's Department tag == resource's Department tag               │
│   AND user's Team tag == resource's Team tag"                      │
│                                                                     │
│  Result: ravi.kumar ──► CAN manage i-abc123 ✅                     │
│          priya.dba  ──► CANNOT manage i-abc123 ❌                  │
└─────────────────────────────────────────────────────────────────────┘
```

### 📝 ABAC Policy Example

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowEC2IfTagsMatch",
      "Effect": "Allow",
      "Action": [
        "ec2:StartInstances",
        "ec2:StopInstances",
        "ec2:RebootInstances",
        "ec2:DescribeInstances"
      ],
      "Resource": "arn:aws:ec2:*:*:instance/*",
      "Condition": {
        "StringEquals": {
          "aws:PrincipalTag/Department": "${aws:ResourceTag/Department}",
          "aws:PrincipalTag/Team": "${aws:ResourceTag/Team}"
        }
      }
    }
  ]
}
```

### 💻 Hands-On: Apply Tags

```bash
# Tag an IAM User
aws iam tag-user \
  --user-name ravi.kumar \
  --tags \
    Key=Department,Value=Engineering \
    Key=Team,Value=DevOps \
    Key=Environment,Value=Production \
    Key=CostCenter,Value=CC-1042

# Tag an IAM Role
aws iam tag-role \
  --role-name EC2-S3-Upload-Role \
  --tags \
    Key=Department,Value=Engineering \
    Key=ManagedBy,Value=Terraform

# Tag an EC2 Instance (for ABAC to work)
aws ec2 create-tags \
  --resources i-1234567890abcdef0 \
  --tags \
    Key=Department,Value=Engineering \
    Key=Team,Value=DevOps

# List tags on a user
aws iam list-user-tags --user-name ravi.kumar

# Remove a tag
aws iam untag-user \
  --user-name ravi.kumar \
  --tag-keys CostCenter
```

### 📌 Tags Best Practices

```
Tagging Strategy for IAM:

Required Tags (enforce via SCP):
├── Environment  → dev | staging | production
├── Department   → engineering | finance | hr
├── Team         → devops | backend | frontend
├── Owner        → email of responsible person
└── CostCenter   → billing code

Optional Tags:
├── Project      → project name
├── ManagedBy    → terraform | cloudformation | console
└── DataClass    → public | internal | confidential

Rules:
├── Max 50 tags per IAM entity
├── Keys are case-sensitive (Department ≠ department)
├── aws: prefix is reserved — don't use it
└── Use consistent naming conventions across teams
```

---

## 9. Hands-On Labs

### 🧪 Lab 1 — Complete IAM Setup from Scratch

```bash
#!/bin/bash
# Lab 1: Full IAM Setup
# Run this in your AWS Free Tier account

echo "========================================="
echo "Lab 1: AWS IAM Complete Setup"
echo "========================================="

# Variables
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
echo "Account ID: $ACCOUNT_ID"

# 1. Create Groups
echo "--- Creating Groups ---"
aws iam create-group --group-name Administrators
aws iam create-group --group-name Developers
aws iam create-group --group-name ReadOnly
aws iam create-group --group-name DBAs

# 2. Attach policies to groups
echo "--- Attaching Policies ---"
aws iam attach-group-policy \
  --group-name Administrators \
  --policy-arn arn:aws:iam::aws:policy/AdministratorAccess

aws iam attach-group-policy \
  --group-name Developers \
  --policy-arn arn:aws:iam::aws:policy/PowerUserAccess

aws iam attach-group-policy \
  --group-name ReadOnly \
  --policy-arn arn:aws:iam::aws:policy/ReadOnlyAccess

aws iam attach-group-policy \
  --group-name DBAs \
  --policy-arn arn:aws:iam::aws:policy/AmazonRDSFullAccess

# 3. Create Users
echo "--- Creating Users ---"
aws iam create-user --user-name admin-user
aws iam create-user --user-name dev-user-1
aws iam create-user --user-name dev-user-2
aws iam create-user --user-name readonly-user

# 4. Add users to groups
echo "--- Adding Users to Groups ---"
aws iam add-user-to-group \
  --group-name Administrators --user-name admin-user

aws iam add-user-to-group \
  --group-name Developers --user-name dev-user-1

aws iam add-user-to-group \
  --group-name Developers --user-name dev-user-2

aws iam add-user-to-group \
  --group-name ReadOnly --user-name readonly-user

# 5. Create console passwords
echo "--- Setting Console Passwords ---"
for user in admin-user dev-user-1 dev-user-2 readonly-user; do
  aws iam create-login-profile \
    --user-name $user \
    --password "Welcome@2024!" \
    --password-reset-required
  echo "Password set for $user"
done

# 6. Create access keys for dev users
echo "--- Creating Access Keys ---"
aws iam create-access-key --user-name dev-user-1

echo "========================================="
echo "✅ Lab 1 Complete!"
echo "Login URL: https://$ACCOUNT_ID.signin.aws.amazon.com/console"
echo "========================================="
```

### 🧪 Lab 2 — Create and Test Custom Policy

```bash
#!/bin/bash
# Lab 2: Custom Policy with S3 Restrictions

echo "Creating custom S3 policy with bucket restriction..."

# Create a test S3 bucket first
BUCKET_NAME="iam-lab-bucket-$(date +%s)"
aws s3 mb s3://$BUCKET_NAME
echo "Created bucket: $BUCKET_NAME"

# Create the policy
cat > restricted-s3-policy.json << EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowSpecificBucket",
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject",
        "s3:DeleteObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::$BUCKET_NAME",
        "arn:aws:s3:::$BUCKET_NAME/*"
      ]
    },
    {
      "Sid": "DenyAllOtherBuckets",
      "Effect": "Deny",
      "Action": "s3:*",
      "NotResource": [
        "arn:aws:s3:::$BUCKET_NAME",
        "arn:aws:s3:::$BUCKET_NAME/*"
      ]
    }
  ]
}
EOF

# Create the policy in AWS
POLICY_ARN=$(aws iam create-policy \
  --policy-name RestrictedS3Access \
  --policy-document file://restricted-s3-policy.json \
  --query 'Policy.Arn' \
  --output text)

echo "Policy created: $POLICY_ARN"

# Attach to user
aws iam attach-user-policy \
  --user-name dev-user-1 \
  --policy-arn $POLICY_ARN

echo "✅ Policy attached to dev-user-1"
echo "Test: dev-user-1 can only access $BUCKET_NAME"
```

### 🧪 Lab 3 — IAM Role for EC2

```bash
#!/bin/bash
# Lab 3: Create and Attach IAM Role to EC2

echo "Creating IAM Role for EC2..."

# Trust policy
cat > ec2-trust.json << 'EOF'
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": { "Service": "ec2.amazonaws.com" },
      "Action": "sts:AssumeRole"
    }
  ]
}
EOF

# Create role
aws iam create-role \
  --role-name MyEC2Role \
  --assume-role-policy-document file://ec2-trust.json

# Attach S3 read access
aws iam attach-role-policy \
  --role-name MyEC2Role \
  --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess

# Create instance profile
aws iam create-instance-profile \
  --instance-profile-name MyEC2Profile

# Add role to instance profile
aws iam add-role-to-instance-profile \
  --instance-profile-name MyEC2Profile \
  --role-name MyEC2Role

echo "✅ EC2 Role created!"
echo "Use 'MyEC2Profile' when launching EC2 instances"

# Verify
aws iam get-role --role-name MyEC2Role
aws iam list-attached-role-policies --role-name MyEC2Role
```

### 🧪 Lab 4 — Security Audit Script

```bash
#!/bin/bash
# Lab 4: IAM Security Audit

echo "============================================"
echo "        AWS IAM Security Audit Report"
echo "============================================"
echo "Date: $(date)"
echo ""

# 1. Check root MFA
echo "--- Root Account Security ---"
aws iam get-account-summary \
  --query 'SummaryMap.{AccountMFAEnabled:AccountMFAEnabled,AccountAccessKeysPresent:AccountAccessKeysPresent}'

# 2. Generate credential report
echo ""
echo "--- Generating Credential Report ---"
aws iam generate-credential-report > /dev/null 2>&1
sleep 3

# Download and parse
aws iam get-credential-report \
  --query 'Content' \
  --output text | base64 -d > /tmp/cred-report.csv

echo "Users with no MFA enabled:"
awk -F',' 'NR>1 && $8=="false" {print "  ⚠️  "$1}' /tmp/cred-report.csv

echo ""
echo "Users with access keys older than 90 days:"
CUTOFF=$(date -d '90 days ago' +%Y-%m-%d 2>/dev/null || date -v-90d +%Y-%m-%d)
awk -F',' -v cutoff="$CUTOFF" 'NR>1 && $14!="N/A" && $14<cutoff {print "  ⚠️  "$1" - Key created: "$14}' /tmp/cred-report.csv

echo ""
echo "Users who never logged in:"
awk -F',' 'NR>1 && $5=="N/A" && $1!="<root_account>" {print "  ℹ️  "$1}' /tmp/cred-report.csv

# 3. List users with admin access
echo ""
echo "--- Users with AdministratorAccess ---"
aws iam list-users --query 'Users[].UserName' --output text | tr '\t' '\n' | while read user; do
  POLICIES=$(aws iam list-attached-user-policies --user-name $user --query 'AttachedPolicies[].PolicyName' --output text 2>/dev/null)
  if echo "$POLICIES" | grep -q "AdministratorAccess"; then
    echo "  🔴 $user has AdministratorAccess!"
  fi
done

echo ""
echo "============================================"
echo "Audit Complete. Review findings above."
echo "============================================"
```

---

## 10. Real-World Scenarios

### 🏦 Scenario 1 — Banking Application (Capital One Style)

```
Architecture: Multi-Environment Banking App
                                        
  ┌─────────────────────────────────────────────────────┐
  │                  AWS Organization                   │
  │                                                     │
  │  ┌──────────┐   ┌──────────┐   ┌──────────┐        │
  │  │  DEV     │   │  STAGING │   │  PROD    │        │
  │  │ Account  │   │ Account  │   │ Account  │        │
  │  │          │   │          │   │          │        │
  │  │Developers│   │QA Team   │   │SRE Only  │        │
  │  │  Full    │   │ReadOnly  │   │Break-     │       │
  │  │  Access  │   │+Deploy   │   │Glass Role│        │
  │  └──────────┘   └──────────┘   └──────────┘        │
  │                                                     │
  │  SCP: No root usage, us-east-1 only, MFA required  │
  └─────────────────────────────────────────────────────┘
```

```yaml
IAM Structure for Banking App:

Groups:
  SRETeam:
    policies: [CloudWatchFullAccess, EC2ReadOnly, S3ReadOnly]
    mfa_required: true

  BackendDevelopers:
    policies: [S3ReadWrite-dev-bucket, LambdaFullAccess-dev, DynamoDB-dev]
    conditions:
      environment: dev only
      region: us-east-1 only

  ProdDeployRole (Role, not Group):
    assumed_by: Jenkins (CI/CD)
    policies: [ECSDeployPolicy, S3-artifact-upload]
    mfa: not applicable (service)
    duration: 1 hour max

  ComplianceAuditors:
    policies: [ReadOnlyAccess, SecurityAuditPolicy]
    conditions:
      mfa_required: true
      allowed_ips: [audit-office-IP]
```

### 🚀 Scenario 2 — Startup (Minimal Team, Maximum Security)

```bash
# Startup IAM Setup — 5 person team, all wearing multiple hats

# Groups
aws iam create-group --group-name founders
aws iam create-group --group-name engineers
aws iam create-group --group-name contractors

# Founders get wide access (startup reality)
aws iam attach-group-policy \
  --group-name founders \
  --policy-arn arn:aws:iam::aws:policy/PowerUserAccess

# Engineers get dev access
aws iam attach-group-policy \
  --group-name engineers \
  --policy-arn arn:aws:iam::aws:policy/AmazonEC2FullAccess
aws iam attach-group-policy \
  --group-name engineers \
  --policy-arn arn:aws:iam::aws:policy/AmazonS3FullAccess

# Contractors — time-limited, read only, specific project only
# Add condition: access expires after project end date
cat > contractor-policy.json << 'EOF'
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:ListBucket"],
      "Resource": "arn:aws:s3:::project-x-bucket/*",
      "Condition": {
        "DateLessThan": {
          "aws:CurrentTime": "2024-12-31T23:59:59Z"
        }
      }
    }
  ]
}
EOF
```

### 🏭 Scenario 3 — Enterprise with Federated Access (SSO)

```
Corporate SSO Flow:
                                       
  Employee      Active          AWS IAM
  Laptop    ──► Directory  ──►  Identity Center  ──►  AWS Account
  (Okta/        (SAML 2.0)     (assigns Role          (temporary
  Browser)                      based on AD group)     credentials)
                                       
  No IAM Users created!
  Permissions managed via AD groups mapped to IAM Roles.
  
  AD Group          →   IAM Role              →  Permissions
  ─────────────         ──────────               ───────────
  AWS-Admins        →   AdministratorRole     →  AdministratorAccess
  AWS-Developers    →   DeveloperRole         →  PowerUserAccess
  AWS-ReadOnly      →   AuditRole             →  ReadOnlyAccess
```

---

## 11. Interview Q&A

### 🎯 Top 20 AWS IAM Interview Questions

---

**Q1: What is the difference between Authentication and Authorization in IAM?**

```
Authentication = Verifying WHO you are
  - Console password proves you're the account owner
  - MFA adds a second proof

Authorization = Determining WHAT you can do
  - IAM policies define allowed actions
  - Default: everything denied unless explicitly allowed
```

---

**Q2: What happens when policies conflict — one Allow and one Deny on the same action?**

```
DENY ALWAYS WINS.

Explicit Deny > Explicit Allow > Default Deny (implicit)

Example:
  Group policy: Allow s3:*
  User policy:  Deny s3:DeleteObject

Result: User CAN do all S3 actions EXCEPT DeleteObject.
```

---

**Q3: What is the difference between IAM User and IAM Role?**

```
IAM User:
- Permanent identity
- Has credentials (password / access keys)
- For humans and long-term service accounts
- Tied to one account

IAM Role:
- Temporary identity
- No permanent credentials
- STS issues temporary creds (15min - 12hours)
- Can be assumed by services, users, other accounts
- Used for EC2, Lambda, cross-account, federation
```

---

**Q4: Can a group contain another group?**

```
NO. Groups can only contain IAM Users.
Groups CANNOT be nested within other groups.

This is a common exam trick question.
```

---

**Q5: What is the principle of Least Privilege?**

```
Grant ONLY the permissions required to perform a task.
Nothing more, nothing less.

Bad:  Give AdministratorAccess to everyone (lazy but risky)
Good: Developer gets S3 read/write only for their bucket
      Lambda gets only DynamoDB read for that table
      Auditor gets ReadOnly with no write/delete
```

---

**Q6: What is STS and when is it used?**

```
AWS Security Token Service (STS):
- Issues temporary security credentials
- Used when assuming IAM Roles
- Credentials include: Access Key, Secret Key, Session Token
- Default duration: 1 hour (min: 15 min, max: 12 hours)

Use cases:
- EC2/Lambda assuming a role
- Cross-account access
- Identity federation (SAML, OIDC)
- AWS CLI: aws sts assume-role
```

---

**Q7: What is a Trust Policy vs Permission Policy in a Role?**

```
Trust Policy (who can assume the role):
{
  "Principal": { "Service": "ec2.amazonaws.com" },
  "Action": "sts:AssumeRole"
}

Permission Policy (what the role can do):
{
  "Effect": "Allow",
  "Action": "s3:*",
  "Resource": "*"
}

Think of it as:
  Trust Policy = Door (who can enter)
  Permission Policy = Keys (what they can do inside)
```

---

**Q8: What are the types of IAM policies?**

```
1. AWS Managed Policies
   - Created and maintained by AWS
   - Common use cases (ReadOnlyAccess, AdministratorAccess)
   
2. Customer Managed Policies
   - Created by you, maintained by you
   - Versioning supported (up to 5 versions)
   - Best for custom, reusable policies
   
3. Inline Policies
   - Embedded directly in a user/group/role
   - 1:1 relationship — deleted when entity deleted
   - Cannot be reused
   - Avoid unless specific coupling is needed
```

---

**Q9: How do you secure the root account?**

```
1. Enable MFA immediately (hardware token recommended)
2. Delete root access keys
3. Never use for daily tasks
4. Store root credentials in secure vault
5. Enable billing alerts
6. Create admin IAM user for administrative tasks
```

---

**Q10: What is ABAC and how is it different from RBAC?**

```
RBAC (Role-Based Access Control):
  - Permissions based on job roles
  - Need to create and manage many different policies
  - Doesn't scale well (100 teams = 100 policies)

ABAC (Attribute-Based Access Control):
  - Permissions based on tags/attributes
  - One policy uses tag conditions
  - Scales automatically (add tag = get access)
  - Modern, used in large enterprises

AWS Implementation:
  - Tag users and resources
  - Use aws:PrincipalTag and aws:ResourceTag in conditions
```

---

**Q11: What is the IAM Credential Report?**

```
A CSV file listing all IAM users and their credential status:
- Password last used
- MFA enabled?
- Access keys active?
- Access key last used
- Access key age

Generate:  aws iam generate-credential-report
Download:  aws iam get-credential-report
Use:       Security audits, compliance, key rotation tracking
```

---

**Q12: What is IAM Access Analyzer?**

```
AWS tool that analyzes resource-based policies and identifies:
- Resources accessible from outside the account
- Public access to S3 buckets, KMS keys, IAM roles
- Cross-account access

Use for:
- Security audits
- Identifying over-permissive access
- Compliance reporting
- Triggered findings → SNS → Auto-remediation
```

---

**Q13: How many access keys can an IAM user have?**

```
Maximum: 2 access keys per user

Why 2? To allow rotation without downtime:
  1. Create new key (key 2 active)
  2. Update applications with new key
  3. Deactivate old key (key 1 inactive)
  4. Monitor for errors
  5. Delete old key
```

---

**Q14: What is an IAM Permission Boundary?**

```
A Permission Boundary sets the MAXIMUM permissions
an IAM entity can have.

Even if a policy grants wide access, the boundary caps it.

Use case: Delegate IAM management safely.
  - Give developers ability to create IAM roles
  - But boundary prevents them from creating roles
    with more permissions than they have

Boundary ∩ Identity Policy = Effective Permissions
```

---

**Q15: What is the difference between Resource-Based Policy and Identity-Based Policy?**

```
Identity-Based Policy:
  - Attached to: IAM User, Group, Role
  - Controls: What the identity can do
  - Example: "This user can access S3"

Resource-Based Policy:
  - Attached to: The resource itself (S3, SQS, KMS, etc.)
  - Controls: Who can access this resource
  - Includes Principal: clause
  - Example: "This S3 bucket allows Account B to read"

Both can be used together for cross-account access.
```

---

## 12. Security Best Practices

### ✅ Complete IAM Security Checklist

```
ACCOUNT LEVEL:
[ ] Enable MFA on root account
[ ] Delete root access keys
[ ] Set account-level password policy
[ ] Enable IAM Access Analyzer
[ ] Enable CloudTrail in all regions
[ ] Enable AWS Config for compliance monitoring
[ ] Use AWS Organizations + SCPs for multi-account

USER MANAGEMENT:
[ ] One IAM user per person (no sharing)
[ ] Enforce MFA for all human users
[ ] Rotate access keys every 90 days
[ ] Delete unused users (run quarterly audit)
[ ] Use groups to assign permissions (not direct user policies)
[ ] Use descriptive, consistent naming (firstname.lastname)

PERMISSIONS:
[ ] Apply least privilege principle
[ ] Start with read-only, expand as needed
[ ] Avoid AdministratorAccess except for break-glass accounts
[ ] Use AWS Managed policies where possible
[ ] Review and remove unused permissions regularly
[ ] Use Permission Boundaries when delegating IAM management

ROLES:
[ ] Use roles instead of long-lived access keys for EC2/Lambda
[ ] Set appropriate session duration (shorter = more secure)
[ ] Require MFA in role trust policies for human users
[ ] Use external ID for cross-account roles (confused deputy prevention)
[ ] Audit assumed roles regularly

MONITORING:
[ ] Review CloudTrail logs for unexpected IAM activity
[ ] Set up CloudWatch alarms for root login
[ ] Run IAM credential report monthly
[ ] Use IAM Access Analyzer findings
[ ] Monitor for access keys not rotated in 90 days
```

### 🚨 Real Incidents and Lessons

```
Incident 1: Access Key Exposed on GitHub
  What happened: Developer pushed .env file with AWS keys
  Time to exploit: 3 minutes (bots scan GitHub 24/7)
  Cost: $50,000 bill in 8 hours from crypto mining
  Prevention:
    ✅ Never store keys in code
    ✅ Use git-secrets or truffleHog pre-commit hooks
    ✅ Use IAM roles instead of keys where possible
    ✅ Enable GuardDuty (detects unusual activity)

Incident 2: Overly Permissive Lambda Role
  What happened: Lambda had AdministratorAccess
  Code had SSRF vulnerability → attacker called metadata endpoint
  Attacker used Lambda role to delete production database
  Prevention:
    ✅ Minimal permissions for Lambda roles
    ✅ No AdministratorAccess on service roles ever
    ✅ Block IMDS v1, use IMDSv2

Incident 3: Shared IAM User for CI/CD
  What happened: team@company.com shared by 5 devs for Jenkins
  One dev left, no one rotated keys
  Ex-employee had access for 6 months after leaving
  Prevention:
    ✅ One service account per CI/CD pipeline
    ✅ Use IAM roles for CI/CD where possible
    ✅ Automated offboarding process
```

---

## 13. Quick Reference Cheat Sheet

### 🃏 IAM Concepts Summary Card

```
┌──────────────────────────────────────────────────────────────────┐
│                    AWS IAM CHEAT SHEET                           │
├────────────────┬─────────────────────────────────────────────────┤
│ Concept        │ Key Points                                      │
├────────────────┼─────────────────────────────────────────────────┤
│ IAM User       │ Permanent | Has creds | 1 per person            │
│ IAM Group      │ Users only | No nesting | Max 10 groups/user    │
│ IAM Policy     │ JSON | Effect+Action+Resource | Deny wins       │
│ IAM Role       │ Temporary | STS | 15min-12hr | No creds         │
│ MFA            │ TOTP/Hardware | Root mandatory | 6-digit code   │
│ Conditions     │ IP/Region/MFA/Time/Tags | Context-aware         │
│ ABAC           │ Tag-based | Scalable | PrincipalTag+ResourceTag  │
│ STS            │ Temp creds | AssumeRole | Session token         │
│ Root Account   │ MFA on! | Delete keys | God mode — avoid        │
│ Least Privilege│ Minimum needed | Start small | Expand later     │
└────────────────┴─────────────────────────────────────────────────┘
```

### 📊 Useful AWS CLI IAM Commands

```bash
# ── USERS ──────────────────────────────────────────────
aws iam list-users                                     # List all users
aws iam get-user --user-name USERNAME                  # User details
aws iam create-user --user-name USERNAME               # Create user
aws iam delete-user --user-name USERNAME               # Delete user
aws iam list-access-keys --user-name USERNAME          # List access keys
aws iam create-access-key --user-name USERNAME         # Create access key
aws iam delete-access-key --user-name USERNAME \
  --access-key-id AKID                                 # Delete access key

# ── GROUPS ─────────────────────────────────────────────
aws iam list-groups                                    # List all groups
aws iam create-group --group-name GROUPNAME            # Create group
aws iam add-user-to-group \                            # Add user to group
  --group-name GROUPNAME --user-name USERNAME
aws iam get-group --group-name GROUPNAME               # List group members

# ── POLICIES ───────────────────────────────────────────
aws iam list-policies --scope Local                    # List custom policies
aws iam create-policy \                                # Create policy
  --policy-name NAME --policy-document file://pol.json
aws iam attach-user-policy \                           # Attach to user
  --user-name USERNAME --policy-arn ARN
aws iam attach-group-policy \                          # Attach to group
  --group-name GROUPNAME --policy-arn ARN
aws iam simulate-principal-policy \                    # Test permissions
  --policy-source-arn ARN \
  --action-names ACTION --resource-arns RESOURCE

# ── ROLES ──────────────────────────────────────────────
aws iam list-roles                                     # List all roles
aws iam create-role \                                  # Create role
  --role-name NAME --assume-role-policy-document file://trust.json
aws iam attach-role-policy \                           # Attach policy
  --role-name NAME --policy-arn ARN
aws sts assume-role \                                  # Assume a role
  --role-arn ARN --role-session-name SESSION

# ── SECURITY ───────────────────────────────────────────
aws iam generate-credential-report                     # Generate report
aws iam get-credential-report                          # Download report
aws iam list-mfa-devices                               # List MFA devices
aws iam get-account-password-policy                    # View pwd policy
aws iam update-account-password-policy \               # Set pwd policy
  --minimum-password-length 14 --require-symbols
```

### 🔗 Official AWS Documentation Links

| Topic | AWS Documentation |
|-------|-------------------|
| IAM Overview | [IAM User Guide](https://docs.aws.amazon.com/IAM/latest/UserGuide/) |
| IAM Policies | [Policy Reference](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies.html) |
| IAM Roles | [Roles Guide](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html) |
| STS | [STS API Reference](https://docs.aws.amazon.com/STS/latest/APIReference/) |
| IAM Best Practices | [Security Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html) |
| ABAC | [ABAC Guide](https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction_attribute-based-access-control.html) |
| Condition Keys | [Global Condition Keys](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html) |
| IAM Access Analyzer | [Analyzer Docs](https://docs.aws.amazon.com/IAM/latest/UserGuide/what-is-access-analyzer.html) |

---

## 📚 Further Learning Path

```
Beginner (Week 1-2):
  ✅ Complete this guide
  ✅ Create Free Tier AWS account
  ✅ Run all hands-on labs
  ✅ Build: Users → Groups → Policies setup

Intermediate (Week 3-4):
  📖 IAM Roles deep dive (EC2, Lambda, Cross-Account)
  📖 S3 Bucket Policies with IAM
  📖 AWS Organizations + SCPs
  📖 AWS CloudTrail + IAM Access Analyzer

Advanced (Month 2):
  📖 AWS IAM Identity Center (SSO)
  📖 SAML/OIDC Federation
  📖 Permission Boundaries
  📖 IAM in Terraform / CloudFormation
  📖 Custom Identity Brokers

Certifications:
  🏅 AWS Cloud Practitioner (CLF-C02) — IAM 15%
  🏅 AWS Solutions Architect Associate (SAA-C03) — IAM 20-25%
  🏅 AWS Security Specialty (SCS-C02) — IAM 40%+
```

---

## 🤝 Contributing

This guide is maintained by **Teja Technologies** for the AWS community.

Found an error? Have a better example? Want to add a use case?

1. Fork this repository
2. Create a branch: `git checkout -b feature/improve-iam-guide`
3. Make your changes
4. Submit a Pull Request

All contributions are welcome — especially real-world scenarios from your projects!

---

## 📢 About the Author

**Teja Balamanasa** — Senior Cloud DevSecOps Engineer

- 12+ years in AWS, DevOps, multi-cloud environments
- Capital One — Cloud Infrastructure Engineering
- Founder, Teja Technologies — Cloud training and education
- AWS Community Builder 🏗️

> *"IAM is not just a service — it is the security foundation of everything you build in AWS. Master this and you master cloud security."*

---

<div align="center">

⭐ **Star this repo** | 🍴 **Fork it** | 📢 **Share with your team**

Made with ❤️ for the AWS Community

</div>

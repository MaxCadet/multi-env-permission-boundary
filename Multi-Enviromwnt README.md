# 🛡️ Multi-Environment Permission Boundary System

> Simulates a real-world company AWS environment with Dev, Staging, and Prod isolation — enforcing IAM permission boundaries so developers can never touch production.

![AWS](https://img.shields.io/badge/AWS-IAM%20%7C%20Organizations%20%7C%20S3%20%7C%20CloudTrail-orange?logo=amazonaws)
![Python](https://img.shields.io/badge/Python-3.12-blue?logo=python)
![License](https://img.shields.io/badge/License-MIT-green)

---

## 📌 Overview

The **Multi-Environment Permission Boundary System** models a company's AWS infrastructure across three isolated environments: **Dev**, **Staging**, and **Prod**. Using IAM permission boundaries, developer roles are strictly scoped to the Dev environment — they can deploy, test, and iterate freely in Dev, but are blocked from modifying or accessing Staging and Prod resources entirely.

This project demonstrates a   core real-world cloud security pattern used by engineering teams to enforce environment separation and prevent accidental or unauthorized changes to production.

---

## 🏗️ Architecture

```
AWS Account
├── Dev Environment
│   ├── IAM Role: DeveloperRole (with permission boundary)
│   ├── S3 Bucket: company-app-dev
│   └── ✅ Developers can deploy here
│
├── Staging Environment
│   ├── IAM Role: StagingDeployRole
│   ├── S3 Bucket: company-app-staging
│   └── ⚠️  Developers blocked — CI/CD only
│
└── Prod Environment
    ├── IAM Role: ProdAdminRole
    ├── S3 Bucket: company-app-prod
    └── 🚫 Developers fully blocked
```

---

## ✨ Features

- **Permission boundaries** — IAM boundaries scoped per environment, preventing privilege escalation
- **Environment isolation** — Dev, Staging, and Prod are separated by role and resource naming conventions
- **Least privilege enforcement** — Developers get only the permissions they need, nothing more
- **Blocked prod access** — Any developer attempt to access Prod resources is denied at the IAM policy level
- **Auditable setup** — All roles and boundaries defined in JSON policy files for transparency and version control

---

## 🛠️ Tech Stack

| Service | Purpose | 
|---|---|
| AWS IAM | Roles, policies, and permission boundaries per environment |
| AWS S3 | Simulated environment resources (dev/staging/prod buckets) |
| AWS CloudTrail | Audit log of all API calls across environments |
| Python (boto3) | Scripts to provision roles, boundaries, and test access |

---

## 🔐 How Permission Boundaries Work Here

A **permission boundary** is an IAM policy attached to a role that sets the *maximum* permissions that role can ever have — even if broader permissions are granted elsewhere.

In this project:

| Role | Permission Boundary | Can Access Dev? | Can Access Staging? | Can Access Prod? |
|---|---|---|---|---|
| `DeveloperRole` | `dev-boundary-policy` | ✅ Yes | ❌ No | ❌ No |
| `StagingDeployRole` | `staging-boundary-policy` | ✅ Yes | ✅ Yes | ❌ No |
| `ProdAdminRole` | None (admin only) | ✅ Yes | ✅ Yes | ✅ Yes |

---

## 🚀 Setup & Deployment

### Prerequisites

- AWS account with admin access
- AWS CLI configured (`aws configure`)
- Python 3.11+

### 1. Clone the repository

```bash
git clone https://github.com/MaxCadet/multi-env-permission-boundary.git
cd multi-env-permission-boundary
```

### 2. Create the simulated environment S3 buckets

```bash
aws s3 mb s3://company-app-dev
aws s3 mb s3://company-app-staging
aws s3 mb s3://company-app-prod
```

### 3. Create the permission boundary policies

```bash
aws iam create-policy \
  --policy-name dev-boundary-policy \
  --policy-document file://policies/dev-boundary.json

aws iam create-policy \
  --policy-name staging-boundary-policy \
  --policy-document file://policies/staging-boundary.json
```

### 4. Create and attach IAM roles

```bash
python scripts/setup_roles.py
```

### 5. Test access enforcement

```bash
python scripts/test_access.py
```

This script simulates each role attempting to access Dev, Staging, and Prod buckets and prints an access matrix showing what is allowed vs. denied.

---

## 📂 Project Structure

```
multi-env-permission-boundary/
├── policies/
│   ├── dev-boundary.json          # Permission boundary for DeveloperRole
│   ├── staging-boundary.json      # Permission boundary for StagingDeployRole
│   ├── developer-role-policy.json # Base permissions for developers
│   └── trust-policy.json          # Trust relationship for role assumption
├── scripts/
│   ├── setup_roles.py             # Provisions all IAM roles and boundaries
│   └── test_access.py             # Validates environment isolation
└── README.md
```

---

## 📋 Sample Permission Boundary (Dev)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject",
        "s3:DeleteObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::company-app-dev",
        "arn:aws:s3:::company-app-dev/*"
      ]
    },
    {
      "Effect": "Deny",
      "Action": "*",
      "Resource": [
        "arn:aws:s3:::company-app-staging",
        "arn:aws:s3:::company-app-staging/*",
        "arn:aws:s3:::company-app-prod",
        "arn:aws:s3:::company-app-prod/*"
      ]
    }
  ]
}
```

---

## 📊 Sample Access Test Output

```
Testing environment access for each role...

Role: DeveloperRole
  ✅ Dev     → Access GRANTED
  ❌ Staging → Access DENIED (boundary restriction)
  ❌ Prod    → Access DENIED (boundary restriction)

Role: StagingDeployRole
  ✅ Dev     → Access GRANTED
  ✅ Staging → Access GRANTED
  ❌ Prod    → Access DENIED (boundary restriction)

Role: ProdAdminRole
  ✅ Dev     → Access GRANTED
  ✅ Staging → Access GRANTED
  ✅ Prod    → Access GRANTED
```

---

## 💡 What I Learned

- How **IAM permission boundaries** differ from standard IAM policies and why both are needed
- How to simulate **multi-environment AWS architecture** within a single account
- How to use **resource-level conditions** to enforce environment separation
- The difference between identity-based policies, resource-based policies, and permission boundaries
- How real engineering teams enforce **dev/staging/prod isolation** in practice

---

## 🔭 Future Improvements

- [ ] Migrate to **AWS Organizations** with separate accounts per environment for stronger isolation
- [ ] Add **Service Control Policies (SCPs)** at the organization level
- [ ] Integrate **AWS Config rules** to continuously monitor for boundary violations
- [ ] Automate provisioning with **Terraform or AWS CDK**

---

## 👤 Author

**Max** — [@MaxCadet](https://github.com/MaxCadet)

<h1>Project-6</h1>

### **Multi-Account Setup with AWS Organizations using Terraform**

#### **Scenario Overview:**
You are setting up a **multi-account AWS environment** using **AWS Organizations** for different teams, with centralized governance, and managing access using **Service Control Policies (SCPs)**.

- AWS Organizations will allow you to centrally manage multiple AWS accounts.
- **Service Control Policies (SCPs)** will enforce governance across the accounts by restricting specific actions.
- This approach can be used to automate account creation for different environments (e.g., Dev, Staging, Prod) or teams.

---

### **Directory Structure**

```
multi-account-setup/
├── main.tf               # Main Terraform config
├── variables.tf          # Variable definitions
├── outputs.tf            # Output values
├── dev/terraform.tfvars  # Dev-specific variables
├── prod/terraform.tfvars # Prod-specific variables
└── modules/
    ├── organizations/
    │   ├── main.tf       # AWS Organizations resources
    │   └── variables.tf  # Organization-specific variables
    └── scp/
        ├── main.tf       # Service Control Policies
        └── variables.tf  # SCP-specific variables
```

---

### **Main Configuration (`main.tf`)**

```hcl
provider "aws" {
  region = var.aws_region
}

# AWS Organization Module
module "organization" {
  source                = "./modules/organizations"
  org_name              = var.org_name
  master_account_email  = var.master_account_email
  service_control_policy = module.scp.policy_arn
}

# SCP Module
module "scp" {
  source         = "./modules/scp"
  scp_name       = var.scp_name
  scp_description = var.scp_description
  scp_policy_document = file(var.scp_policy_file)
}

# Create Organizational Units (OUs)
resource "aws_organizations_organizational_unit" "dev" {
  name      = "Dev"
  parent_id = module.organization.root_id
}

resource "aws_organizations_organizational_unit" "prod" {
  name      = "Prod"
  parent_id = module.organization.root_id
}

# Create Accounts in OUs
resource "aws_organizations_account" "dev_account" {
  name      = "Dev Account"
  email     = var.dev_account_email
  role_name = "OrganizationAccountAccessRole"
  organizational_unit_id = aws_organizations_organizational_unit.dev.id
}

resource "aws_organizations_account" "prod_account" {
  name      = "Prod Account"
  email     = var.prod_account_email
  role_name = "OrganizationAccountAccessRole"
  organizational_unit_id = aws_organizations_organizational_unit.prod.id
}
```

---

### **AWS Organizations Module (`modules/organizations/main.tf`)**

```hcl
resource "aws_organizations_organization" "this" {
  aws_service_access_principals = ["config.amazonaws.com", "cloudtrail.amazonaws.com"]
  feature_set                   = "ALL"
}

output "root_id" {
  value = aws_organizations_organization.this.roots[0].id
}
```

---

### **SCP Module (`modules/scp/main.tf`)**

```hcl
resource "aws_organizations_policy" "scp" {
  name        = var.scp_name
  description = var.scp_description
  content     = var.scp_policy_document
  type        = "SERVICE_CONTROL_POLICY"
}

output "policy_arn" {
  value = aws_organizations_policy.scp.arn
}
```

---

### **Variables (`variables.tf`)**

```hcl
variable "aws_region" {
  description = "The AWS region to deploy resources"
  default     = "us-east-1"
}

# Organization variables
variable "org_name" {
  description = "Name of the AWS Organization"
}

variable "master_account_email" {
  description = "Email for the master AWS account"
}

# SCP variables
variable "scp_name" {}
variable "scp_description" {}
variable "scp_policy_file" {}

# Accounts emails
variable "dev_account_email" {}
variable "prod_account_email" {}
```

---

### **Sample Service Control Policy (SCP) - Restrict S3 Access (`scp-policy.json`)**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": "s3:*",
      "Resource": "*"
    }
  ]
}
```

This SCP will restrict **all S3 actions** for the accounts attached to this policy.

---

### **Environment-Specific Variables (`dev/terraform.tfvars` and `prod/terraform.tfvars`)**

**`dev/terraform.tfvars`**

```hcl
org_name             = "MyOrganization"
master_account_email = "master-dev@example.com"
dev_account_email    = "dev-team@example.com"
scp_name             = "DenyS3Access"
scp_description      = "Restricts S3 access for all accounts"
scp_policy_file      = "./scp-policy.json"
```

**`prod/terraform.tfvars`**

```hcl
org_name             = "MyOrganization"
master_account_email = "master-prod@example.com"
prod_account_email   = "prod-team@example.com"
scp_name             = "DenyS3Access"
scp_description      = "Restricts S3 access for all accounts"
scp_policy_file      = "./scp-policy.json"
```

---

### **How to Apply:**

1. **Initialize Terraform:**

   ```bash
   terraform init
   ```

2. **Apply the Dev Environment:**

   ```bash
   terraform apply -var-file="dev/terraform.tfvars"
   ```

3. **Apply the Prod Environment:**

   ```bash
   terraform apply -var-file="prod/terraform.tfvars"
   ```

---

### **Explanation:**

- **AWS Organizations**: Automates the creation of an organization and multiple accounts. The root of the organization and the child **Organizational Units (OUs)** (Dev, Prod) are created to segregate environments.
- **Service Control Policies (SCPs)**: An SCP restricts actions for all member accounts in the organization (in this case, disabling S3 access). The policy is created and applied across the relevant organizational units (Dev, Prod).
- **New Accounts**: Automatically provisions new AWS accounts under the relevant organizational unit (Dev or Prod) using the email provided.

This setup provides **centralized governance**, while automating the creation of multiple AWS accounts with restricted actions using SCPs. It’s ideal for managing environments across various teams in a **multi-account AWS architecture**.

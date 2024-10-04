<h1>Project-6</h1>

### **Project: Infrastructure as Code (IaC) with Terraform and GitHub**

The goal of this project is to manage cloud infrastructure (AWS in this example) using **Terraform** and **GitHub**. The deployment is automated via a **CI/CD pipeline** set up using **GitHub Actions** to apply infrastructure changes. We will use Terraform to provision resources such as EC2 instances, VPCs, S3 buckets, etc., and automate the process through GitHub workflows.

---

### **1. Project Directory Structure**

```bash
iac-terraform/
│
├── .github/
│   └── workflows/
│       └── terraform-deployment.yml      # GitHub Actions workflow file
│
├── terraform/
│   ├── main.tf                           # Terraform configuration file
│   ├── variables.tf                      # Variables used in Terraform
│   ├── outputs.tf                        # Output values
│   ├── provider.tf                       # Cloud provider configuration
│   └── terraform.tfvars                  # Variable values (environment-specific)
│
├── .gitignore                            # Git ignore file
└── README.md                             # Project documentation
```

### **2. Project Files Content**

#### **2.1. terraform/main.tf**

This Terraform file defines the cloud infrastructure. For simplicity, we are creating an EC2 instance in AWS:

```hcl
provider "aws" {
  region = var.aws_region
}

resource "aws_instance" "my_ec2" {
  ami           = var.ami_id
  instance_type = var.instance_type

  tags = {
    Name = "MyTerraformEC2"
  }
}
```

#### **2.2. terraform/variables.tf**

Define variables used across the Terraform configuration:

```hcl
variable "aws_region" {
  description = "The AWS region where resources will be deployed"
  type        = string
}

variable "ami_id" {
  description = "AMI ID for the EC2 instance"
  type        = string
}

variable "instance_type" {
  description = "Instance type for EC2"
  type        = string
}
```

#### **2.3. terraform/outputs.tf**

Defines the output values you might need after deploying resources:

```hcl
output "instance_id" {
  description = "ID of the EC2 instance"
  value       = aws_instance.my_ec2.id
}

output "instance_public_ip" {
  description = "Public IP of the EC2 instance"
  value       = aws_instance.my_ec2.public_ip
}
```

#### **2.4. terraform/provider.tf**

Specifies the cloud provider configuration, in this case, **AWS**:

```hcl
provider "aws" {
  region = var.aws_region
}
```

#### **2.5. terraform/terraform.tfvars**

Provide values for the variables defined in `variables.tf`. You can create environment-specific `terraform.tfvars` files (e.g., `dev`, `staging`, `prod`) to differentiate between deployments:

```hcl
aws_region    = "us-east-1"
ami_id        = "ami-0c55b159cbfafe1f0"
instance_type = "t2.micro"
```

#### **2.6. .github/workflows/terraform-deployment.yml**

This is the GitHub Actions workflow file that automatically applies the Terraform changes when code is pushed to the repository:

```yaml
name: Terraform Deployment

on:
  push:
    branches:
      - main
      - dev
      - staging
      - prod

jobs:
  terraform:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Set up Terraform
        uses: hashicorp/setup-terraform@v2

      - name: Terraform Init
        run: terraform init
        working-directory: ./terraform

      - name: Terraform Plan
        run: terraform plan
        working-directory: ./terraform

      - name: Terraform Apply
        if: github.ref == 'refs/heads/main' || github.ref == 'refs/heads/staging' || github.ref == 'refs/heads/prod'
        run: terraform apply -auto-approve
        working-directory: ./terraform
```

This workflow:
- Runs on pushes to the `main`, `dev`, `staging`, and `prod` branches.
- Executes Terraform `init`, `plan`, and `apply` commands.

> **Note:** Only the `main`, `staging`, and `prod` branches trigger `terraform apply` to make infrastructure changes.

#### **2.7. .gitignore**

```bash
# Ignore Terraform state files and backups
*.tfstate
*.tfstate.backup
.terraform/

# Ignore environment variables and sensitive data
*.tfvars
```

#### **2.8. README.md**

```markdown
# Infrastructure as Code (IaC) with Terraform

## Overview

This project demonstrates how to use Terraform for managing AWS infrastructure, integrated with a CI/CD pipeline using GitHub Actions. Terraform provisions an EC2 instance, and GitHub Actions automatically applies changes upon code push to the repository.

## Setup

### Prerequisites

- AWS Account with credentials (access key and secret key).
- Terraform installed locally.
- GitHub repository set up.

### Local Setup

1. Clone the repository:

   ```bash
   git clone https://github.com/your-username/iac-terraform.git
   cd iac-terraform
   ```

2. Initialize and apply Terraform locally:

   ```bash
   cd terraform
   terraform init
   terraform plan
   terraform apply
   ```

### CI/CD Pipeline

The pipeline is triggered by a push to the `main`, `dev`, `staging`, or `prod` branches:

- **main**: Production infrastructure.
- **dev**: Development infrastructure.
- **staging**: Staging infrastructure.
  
The CI/CD pipeline automatically applies infrastructure changes using Terraform. Before applying any changes, `terraform plan` is run to preview the changes.

### GitHub Secrets

To securely configure AWS credentials in GitHub Actions, add the following secrets:

- `AWS_ACCESS_KEY_ID`: Your AWS access key.
- `AWS_SECRET_ACCESS_KEY`: Your AWS secret key.
```

---

### **3. Steps to Save and Deploy the Project**

1. **Create a New GitHub Repository**: Create a new GitHub repository for your Terraform project.
2. **Clone the Repository**: Clone your repository locally.

   ```bash
   git clone https://github.com/your-username/iac-terraform.git
   cd iac-terraform
   ```

3. **Create the Directory Structure**: Create the directories and files based on the structure provided.
4. **Add Content to Files**: Copy and paste the contents into the respective files.
5. **Commit and Push to GitHub**:

   ```bash
   git add .
   git commit -m "Initial setup of Terraform project with GitHub Actions"
   git push -u origin main
   ```

6. **Configure GitHub Secrets**:
   - Go to your GitHub repository settings.
   - Add the following secrets:
     - `AWS_ACCESS_KEY_ID`: Your AWS access key.
     - `AWS_SECRET_ACCESS_KEY`: Your AWS secret key.

7. **Test the Pipeline**: Push changes to the repository and observe the CI/CD pipeline automatically applying the Terraform configuration.

---

### **4. How the CI/CD Pipeline Works**

- **Trigger Events**: The pipeline is triggered by pushes to the `main`, `dev`, `staging`, or `prod` branches.
- **Terraform Workflow**:
  1. **terraform init**: Initializes Terraform and downloads the required providers.
  2. **terraform plan**: Creates a plan of the changes to be applied.
  3. **terraform apply**: Automatically applies the changes to your AWS infrastructure in the `main`, `staging`, or `prod` environments.

This setup helps maintain cloud infrastructure using a **GitOps** approach, where infrastructure updates are managed through version control. Each environment (`dev`, `staging`, `prod`) has its own branch, making it easier to manage different stages of your infrastructure.

Let me know if you need further assistance or clarification!

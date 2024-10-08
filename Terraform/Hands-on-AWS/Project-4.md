### **Project: Infrastructure as Code (IaC) for CI/CD Pipelines with Terraform**

In this project, you’ll use Terraform to automate the deployment of the infrastructure needed to set up a Jenkins-based **CI/CD pipeline** in AWS. This includes provisioning **Jenkins servers**, setting up **AWS CodeBuild** or **CodePipeline**, creating **S3 buckets** for artifact storage, and automating the creation of **IAM roles and policies** for your CI/CD pipeline.

---

### **1. Project Directory Structure**

Here’s a suggested directory structure for organizing Terraform files and modules specific to the CI/CD infrastructure:

```bash
jenkins-ci-cd-infrastructure/
│
├── modules/
│   ├── ec2-instance/                  # Module for Jenkins EC2 instance
│   ├── s3-bucket/                     # Module for S3 bucket (artifact storage)
│   ├── iam-roles/                     # Module for IAM roles and policies
│   ├── codepipeline/                  # Module for AWS CodePipeline/CodeBuild
│
├── dev/                               # Environment-specific configuration (dev)
│   ├── main.tf                        # Dev environment configurations
│   ├── variables.tf                   # Variables specific to dev
│   └── terraform.tfvars               # Environment-specific variables
├── prod/                              # Environment-specific configuration (prod)
│   ├── main.tf                        # Prod environment configurations
│   ├── variables.tf                   # Variables specific to prod
│   └── terraform.tfvars               # Environment-specific variables
├── provider.tf                        # AWS provider configurations
├── outputs.tf                         # Global outputs for the infrastructure
├── variables.tf                       # Global variables
└── terraform.tfvars                   # Global variable values
```

---

### **2. Terraform Module for Jenkins EC2 Instance**

The Jenkins server will be deployed on an EC2 instance, with Terraform managing the infrastructure.

#### **2.1. `main.tf` (Jenkins EC2 Instance)**

```hcl
resource "aws_instance" "jenkins" {
  ami           = var.jenkins_ami
  instance_type = var.instance_type
  key_name      = var.key_pair
  subnet_id     = var.subnet_id
  security_groups = [var.security_group_id]

  tags = {
    Name = "Jenkins-Server"
  }

  user_data = <<-EOF
    #!/bin/bash
    sudo yum update -y
    sudo yum install -y java-1.8.0-openjdk
    wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
    rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
    yum install jenkins -y
    systemctl enable jenkins
    systemctl start jenkins
  EOF
}
```

#### **2.2. `variables.tf` (EC2 Input Variables)**

```hcl
variable "jenkins_ami" {
  description = "AMI ID for Jenkins server"
  type        = string
}

variable "instance_type" {
  description = "EC2 instance type for Jenkins server"
  type        = string
}

variable "key_pair" {
  description = "SSH key pair for Jenkins server access"
  type        = string
}

variable "subnet_id" {
  description = "Subnet ID for Jenkins instance"
  type        = string
}

variable "security_group_id" {
  description = "Security group for Jenkins instance"
  type        = string
}
```

#### **2.3. `outputs.tf` (EC2 Output)**

```hcl
output "jenkins_public_ip" {
  description = "Public IP address of Jenkins server"
  value       = aws_instance.jenkins.public_ip
}
```

---

### **3. Terraform Module for S3 Bucket (Artifact Storage)**

The S3 bucket will store build artifacts created by the Jenkins pipeline or AWS CodeBuild.

#### **3.1. `main.tf` (S3 Bucket for Artifacts)**

```hcl
resource "aws_s3_bucket" "artifacts" {
  bucket = "${var.bucket_name}-artifacts"
  acl    = "private"

  versioning {
    enabled = true
  }

  tags = {
    Name = "${var.bucket_name}-artifacts"
  }
}

resource "aws_s3_bucket_policy" "artifacts_policy" {
  bucket = aws_s3_bucket.artifacts.id

  policy = <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowJenkinsAccess",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "${aws_s3_bucket.artifacts.arn}/*"
    }
  ]
}
EOF
}
```

#### **3.2. `variables.tf` (S3 Input Variables)**

```hcl
variable "bucket_name" {
  description = "Base name of the S3 bucket"
  type        = string
}
```

---

### **4. Terraform Module for IAM Roles and Policies**

IAM roles are essential for providing Jenkins and the AWS CodePipeline access to deploy and manage infrastructure.

#### **4.1. `main.tf` (IAM Roles for Jenkins and CodePipeline)**

```hcl
resource "aws_iam_role" "jenkins_role" {
  name = "jenkins-role"

  assume_role_policy = <<EOF
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
}

resource "aws_iam_policy" "jenkins_policy" {
  name        = "jenkins-policy"
  description = "Policy for Jenkins access to S3 and EC2"
  
  policy = <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:*",
        "ec2:*"
      ],
      "Resource": "*"
    }
  ]
}
EOF
}

resource "aws_iam_role_policy_attachment" "jenkins_policy_attach" {
  role       = aws_iam_role.jenkins_role.name
  policy_arn = aws_iam_policy.jenkins_policy.arn
}
```

---

### **5. Terraform Module for AWS CodePipeline/CodeBuild**

This module automates setting up **AWS CodePipeline** to integrate with GitHub and Jenkins for continuous integration and deployment.

#### **5.1. `main.tf` (AWS CodePipeline)**

```hcl
resource "aws_codepipeline" "jenkins_pipeline" {
  name = "Jenkins-CodePipeline"

  artifact_store {
    type     = "S3"
    location = aws_s3_bucket.artifacts.bucket
  }

  stage {
    name = "Source"
    action {
      name             = "GitHub_Source"
      category         = "Source"
      owner            = "ThirdParty"
      provider         = "GitHub"
      version          = "1"
      output_artifacts = ["source_output"]

      configuration = {
        Owner  = var.github_owner
        Repo   = var.github_repo
        Branch = var.github_branch
        OAuthToken = var.github_oauth_token
      }
    }
  }

  stage {
    name = "Build"
    action {
      name             = "Jenkins_Build"
      category         = "Build"
      owner            = "AWS"
      provider         = "CodeBuild"
      version          = "1"
      input_artifacts  = ["source_output"]
      output_artifacts = ["build_output"]

      configuration = {
        ProjectName = aws_codebuild_project.jenkins_project.name
      }
    }
  }
}
```

#### **5.2. `variables.tf` (CodePipeline Input Variables)**

```hcl
variable "github_owner" {
  description = "GitHub owner of the repository"
  type        = string
}

variable "github_repo" {
  description = "GitHub repository name"
  type        = string
}

variable "github_branch" {
  description = "Branch to build from GitHub"
  type        = string
}

variable "github_oauth_token" {
  description = "OAuth token for GitHub access"
  type        = string
  sensitive   = true
}
```

Here’s the **single-line format** for **dev** and **prod** environment files, keeping things compact while still illustrating the Terraform structure.

---

### **`dev/main.tf` (Development Environment)**

```hcl
module "jenkins" {
  source           = "../modules/ec2-instance"
  jenkins_ami      = "ami-xxxxxxxx"
  instance_type    = "t2.micro"
  key_pair         = "dev-key"
  subnet_id        = "subnet-123456"
  security_group_id = "sg-123456"
}

module "artifacts" {
  source      = "../modules/s3-bucket"
  bucket_name = "dev-jenkins-artifacts"
}

module "iam" {
  source = "../modules/iam-roles"
}

module "codepipeline" {
  source            = "../modules/codepipeline"
  github_owner      = "your-github-username"
  github_repo       = "your-repo"
  github_branch     = "dev"
  github_oauth_token = "your-token"
}
```

### **`prod/main.tf` (Production Environment)**

```hcl
module "jenkins" {
  source           = "../modules/ec2-instance"
  jenkins_ami      = "ami-xxxxxxxx"
  instance_type    = "t3.medium"
  key_pair         = "prod-key"
  subnet_id        = "subnet-654321"
  security_group_id = "sg-654321"
}

module "artifacts" {
  source      = "../modules/s3-bucket"
  bucket_name = "prod-jenkins-artifacts"
}

module "iam" {
  source = "../modules/iam-roles"
}

module "codepipeline" {
  source            = "../modules/codepipeline"
  github_owner      = "your-github-username"
  github_repo       = "your-repo"
  github_branch     = "main"
  github_oauth_token = "your-token"
}
```

---

### **`dev/terraform.tfvars` (Development Variables)**

```hcl
jenkins_ami = "ami-xxxxxxxx"
instance_type = "t2.micro"
key_pair = "dev-key"
subnet_id = "subnet-123456"
security_group_id = "sg-123456"
bucket_name = "dev-jenkins-artifacts"
github_branch = "dev"
```

### **`prod/terraform.tfvars` (Production Variables)**

```hcl
jenkins_ami = "ami-xxxxxxxx"
instance_type = "t3.medium"
key_pair = "prod-key"
subnet_id = "subnet-654321"
security_group_id = "sg-654321"
bucket_name = "prod-jenkins-artifacts"
github_branch = "main"
```

---

These **environment-specific configurations** (development and production) help you manage separate infrastructure deployments by switching between environments simply using the appropriate variable files (`dev/terraform.tfvars` and `prod/terraform.tfvars`).

You can run these with:

- **Dev**:

  ```bash
  terraform apply -var-file="dev/terraform.tfvars"
  ```

- **Prod**:

  ```bash
  terraform apply -var-file="prod/terraform.tfvars"
  ```

This keeps the core infrastructure (modules) the same but allows environment-specific customization!
---

### **6. Applying the Terraform Configuration**

1. **Initialize Terraform**:

   ```bash
   terraform init
   ```

2. **Plan the Infrastructure**:

   ```bash
   terraform plan -var-file="dev/terraform.tfvars"
   ```

3. **Apply the Infrastructure**:

   ```bash
   terraform apply -var-file="dev/terraform.tfvars"
   ```

---

This project automates the setup of a Jenkins-based CI/CD pipeline with the help of **AWS CodePipeline**, **CodeBuild**, **S3 for artifact storage**, and appropriate **IAM roles and policies** using Terraform. This infrastructure is scalable and can be extended with additional features like **auto-scaling** for the Jenkins server or **Lambda functions** for post-deployment hooks.

Let me know if you need more details or specific configuration customizations!

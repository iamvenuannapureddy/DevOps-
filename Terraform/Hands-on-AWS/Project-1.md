<h1>Project-1</h1>
### **Project: Provisioning AWS EC2 Instances Using Terraform**

This project involves provisioning multiple AWS EC2 instances across different environments (dev, test, prod) using Terraform. The goal is to create reusable Terraform modules that allow for easy configuration and scaling of EC2 instances by specifying instance types, AMI IDs, key pairs, and user data scripts.

---

### **1. Project Directory Structure**

Here’s a directory structure for organizing your Terraform files and modules:

```bash
aws-ec2-provisioning/
│
├── modules/
│   └── ec2-instance/                 # Reusable module for EC2 instance
│       ├── main.tf                   # Main configuration for the EC2 instance
│       ├── variables.tf              # Input variables for module
│       ├── outputs.tf                # Outputs for module
│       └── user_data.sh              # User data script for instance initialization
├── dev/                              # Terraform configurations for dev environment
│   ├── main.tf                       # Dev environment configurations
│   ├── variables.tf                  # Input variables specific to dev
│   └── terraform.tfvars              # Environment-specific variables
├── test/                             # Terraform configurations for test environment
│   ├── main.tf                       # Test environment configurations
│   ├── variables.tf                  # Input variables specific to test
│   └── terraform.tfvars              # Environment-specific variables
├── prod/                             # Terraform configurations for prod environment
│   ├── main.tf                       # Prod environment configurations
│   ├── variables.tf                  # Input variables specific to prod
│   └── terraform.tfvars              # Environment-specific variables
├── main.tf                           # Root Terraform file to call modules
├── provider.tf                       # Provider configuration for AWS
├── outputs.tf                        # Outputs for overall infrastructure
├── variables.tf                      # Global variables
└── terraform.tfvars                  # Global variable values
```

---

### **2. Terraform EC2 Instance Module**

In the `modules/ec2-instance/` directory, create a reusable Terraform module that provisions EC2 instances.

#### **2.1. `main.tf` (Module Configuration)**

This file provisions the EC2 instance with configurable AMI, instance type, and user data:

```hcl
resource "aws_instance" "this" {
  ami           = var.ami_id
  instance_type = var.instance_type
  key_name      = var.key_name
  user_data     = file("${path.module}/user_data.sh")

  tags = {
    Name = var.instance_name
    Environment = var.environment
  }
}
```

#### **2.2. `variables.tf` (Module Input Variables)**

Define input variables to make the EC2 instance configuration reusable:

```hcl
variable "ami_id" {
  description = "AMI ID for the EC2 instance"
  type        = string
}

variable "instance_type" {
  description = "EC2 instance type (e.g., t2.micro, t2.large)"
  type        = string
}

variable "key_name" {
  description = "Key pair name for SSH access"
  type        = string
}

variable "instance_name" {
  description = "Name tag for the instance"
  type        = string
}

variable "environment" {
  description = "Environment (dev, test, prod)"
  type        = string
}
```

#### **2.3. `outputs.tf` (Module Outputs)**

Capture important information about the EC2 instance, such as its public IP and instance ID:

```hcl
output "instance_id" {
  description = "ID of the EC2 instance"
  value       = aws_instance.this.id
}

output "public_ip" {
  description = "Public IP address of the EC2 instance"
  value       = aws_instance.this.public_ip
}
```

#### **2.4. `user_data.sh` (User Data Script)**

This script is passed to the EC2 instance on launch to automate initialization tasks:

```bash
#!/bin/bash
# Example user data script to install Apache server

sudo yum update -y
sudo yum install -y httpd
sudo systemctl start httpd
sudo systemctl enable httpd
echo "Hello from $(hostname)" > /var/www/html/index.html
```

---

### **3. Environment-Specific Configurations (dev, test, prod)**

Each environment (dev, test, prod) has its own directory with environment-specific configurations. These configurations use the reusable module to provision instances with different parameters.

#### **3.1. `dev/main.tf` (Dev Environment Configuration)**

```hcl
module "ec2_instance_dev" {
  source          = "../modules/ec2-instance"
  ami_id          = var.ami_id
  instance_type   = var.instance_type
  key_name        = var.key_name
  instance_name   = "dev-instance"
  environment     = "dev"
}
```

#### **3.2. `dev/terraform.tfvars` (Dev Environment Variables)**

```hcl
ami_id        = "ami-0123456789abcdef0"
instance_type = "t2.micro"
key_name      = "my-dev-key"
```

Repeat similar configurations for `test` and `prod` environments by adjusting instance types, AMI IDs, and key names as needed.

---

### **4. Provider Configuration**

In the root directory, configure the AWS provider to interact with your AWS account.

#### **`provider.tf`**

```hcl
provider "aws" {
  region = "us-east-1"
}
```

---

### **5. Applying Terraform Configuration**

1. **Initialize Terraform**: Run this command to initialize your working directory and download necessary providers:

   ```bash
   terraform init
   ```

2. **Plan the Infrastructure**: Review what Terraform will create:

   ```bash
   terraform plan -var-file="dev/terraform.tfvars"
   ```

3. **Apply the Changes**: Provision the EC2 instance in the development environment:

   ```bash
   terraform apply -var-file="dev/terraform.tfvars"
   ```

4. **Provision Other Environments**: Apply the same process for test and prod by switching to their respective directories and running the `apply` command.

---

### **6. Using Multiple Environments**

Each environment has its own configuration, allowing you to spin up different EC2 instances for dev, test, and production. By using the same reusable EC2 module across environments, you ensure consistency while customizing instances for each environment.

For example:
- **Development**: Use smaller, cheaper instances (e.g., `t2.micro`).
- **Testing**: Use larger instances (e.g., `t2.medium`) to simulate more realistic workloads.
- **Production**: Use high-performance instances (e.g., `m5.large`) for handling real-world traffic.

---

### **7. Conclusion**

This Terraform project allows you to automate the provisioning of AWS EC2 instances across different environments using reusable modules. With customizable configurations for each environment (dev, test, prod), you can maintain consistency while optimizing resources based on the environment's requirements.

By using this structure, your infrastructure code is modular, maintainable, and scalable. Let me know if you need help with specific configurations or additional details!

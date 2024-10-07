<h1>Project-3</h1>

### **Project: Managing VPC, Subnets, and Networking with Terraform**

In this project, you will create a **secure, isolated network** for your applications using Terraform. The setup will include a **VPC**, **subnets** (public and private), **route tables**, and **security groups**. You’ll also automate the creation of **peering connections**, **VPNs**, or **NAT Gateways** for secure access to the internet and private networks.

---

### **1. Project Directory Structure**

Here’s a suggested directory structure for organizing your Terraform files and reusable modules for networking:

```bash
aws-vpc-networking/
│
├── modules/
│   ├── vpc/                         # Module for VPC, subnets, and route tables
│   ├── security-group/               # Module for security groups
│   └── nat-gateway/                  # Module for NAT Gateway and Elastic IP
│
├── dev/                              # Configuration for dev environment
│   ├── main.tf                       # Dev environment configurations
│   ├── variables.tf                  # Variables specific to dev
│   └── terraform.tfvars              # Environment-specific variables
├── test/                             # Configuration for test environment
│   ├── main.tf                       # Test environment configurations
│   ├── variables.tf                  # Variables specific to test
│   └── terraform.tfvars              # Environment-specific variables
├── prod/                             # Configuration for prod environment
│   ├── main.tf                       # Prod environment configurations
│   ├── variables.tf                  # Variables specific to prod
│   └── terraform.tfvars              # Environment-specific variables
├── provider.tf                       # Provider configuration for AWS
├── outputs.tf                        # Global outputs for the infrastructure
├── variables.tf                      # Global variables
└── terraform.tfvars                  # Global variable values
```

---

### **2. Terraform Module for VPC**

The **VPC** module will create the network, subnets, and routing. It will define private and public subnets across multiple availability zones (AZs).

#### **2.1. `main.tf` (VPC, Subnets, and Route Tables)**

```hcl
resource "aws_vpc" "main" {
  cidr_block = var.vpc_cidr
  enable_dns_support = true
  enable_dns_hostnames = true
  tags = {
    Name = var.vpc_name
  }
}

resource "aws_subnet" "public" {
  count = length(var.public_subnet_cidrs)

  vpc_id            = aws_vpc.main.id
  cidr_block        = var.public_subnet_cidrs[count.index]
  map_public_ip_on_launch = true
  availability_zone = element(var.azs, count.index)
  
  tags = {
    Name = "${var.vpc_name}-public-${count.index}"
  }
}

resource "aws_subnet" "private" {
  count = length(var.private_subnet_cidrs)

  vpc_id            = aws_vpc.main.id
  cidr_block        = var.private_subnet_cidrs[count.index]
  availability_zone = element(var.azs, count.index)

  tags = {
    Name = "${var.vpc_name}-private-${count.index}"
  }
}

resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main.id
  }

  tags = {
    Name = "${var.vpc_name}-public-rt"
  }
}

resource "aws_route_table_association" "public" {
  count          = length(var.public_subnet_cidrs)
  subnet_id      = aws_subnet.public[count.index].id
  route_table_id = aws_route_table.public.id
}

resource "aws_route_table" "private" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_nat_gateway.main.id
  }

  tags = {
    Name = "${var.vpc_name}-private-rt"
  }
}

resource "aws_route_table_association" "private" {
  count          = length(var.private_subnet_cidrs)
  subnet_id      = aws_subnet.private[count.index].id
  route_table_id = aws_route_table.private.id
}
```

#### **2.2. `variables.tf` (VPC Input Variables)**

```hcl
variable "vpc_cidr" {
  description = "CIDR block for the VPC"
  type        = string
}

variable "vpc_name" {
  description = "Name of the VPC"
  type        = string
}

variable "public_subnet_cidrs" {
  description = "List of CIDR blocks for public subnets"
  type        = list(string)
}

variable "private_subnet_cidrs" {
  description = "List of CIDR blocks for private subnets"
  type        = list(string)
}

variable "azs" {
  description = "List of availability zones"
  type        = list(string)
}
```

#### **2.3. `outputs.tf` (VPC Outputs)**

```hcl
output "vpc_id" {
  description = "ID of the VPC"
  value       = aws_vpc.main.id
}

output "public_subnets" {
  description = "IDs of the public subnets"
  value       = aws_subnet.public[*].id
}

output "private_subnets" {
  description = "IDs of the private subnets"
  value       = aws_subnet.private[*].id
}
```

---

### **3. Terraform Module for Security Groups**

The **Security Group** module will define network access rules for your instances.

#### **3.1. `main.tf` (Security Groups)**

```hcl
resource "aws_security_group" "public_sg" {
  vpc_id = var.vpc_id

  ingress {
    description = "Allow HTTP"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    description = "Allow HTTPS"
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "${var.vpc_name}-public-sg"
  }
}

resource "aws_security_group" "private_sg" {
  vpc_id = var.vpc_id

  ingress {
    description = "Allow internal traffic"
    from_port   = 0
    to_port     = 65535
    protocol    = "tcp"
    cidr_blocks = var.private_subnet_cidrs
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "${var.vpc_name}-private-sg"
  }
}
```

#### **3.2. `variables.tf` (Security Group Input Variables)**

```hcl
variable "vpc_id" {
  description = "VPC ID"
  type        = string
}

variable "private_subnet_cidrs" {
  description = "CIDR blocks for private subnets"
  type        = list(string)
}

variable "vpc_name" {
  description = "Name of the VPC"
  type        = string
}
```

---

### **4. Terraform Module for NAT Gateway**

The **NAT Gateway** module will allow instances in private subnets to access the internet securely.

#### **4.1. `main.tf` (NAT Gateway and Elastic IP)**

```hcl
resource "aws_eip" "nat" {
  vpc = true
}

resource "aws_nat_gateway" "main" {
  allocation_id = aws_eip.nat.id
  subnet_id     = aws_subnet.public[0].id
}
```

---

### **5. Peering Connections, VPNs, and Additional Networking**

You can automate the creation of peering connections, VPNs, or other networking features. For example, for VPC peering:

#### **5.1. `main.tf` (VPC Peering)**

```hcl
resource "aws_vpc_peering_connection" "peer" {
  vpc_id        = aws_vpc.main.id
  peer_vpc_id   = var.peer_vpc_id
  auto_accept   = true
  peer_owner_id = var.peer_owner_id

  tags = {
    Name = "vpc-peering"
  }
}

resource "aws_route" "peer_route" {
  route_table_id         = aws_route_table.private.id
  destination_cidr_block = var.peer_cidr
  vpc_peering_connection_id = aws_vpc_peering_connection.peer.id
}
```

---

### **6. Applying Terraform Configuration**

1. **Initialize Terraform**:

   ```bash
   terraform init
   ```

2. **Plan the Infrastructure**:

   ```bash
   terraform plan -var-file="dev/terraform.tfvars"
   ```

3. **Apply the Infrastructure

**:

   ```bash
   terraform apply -var-file="dev/terraform.tfvars"
   ```

---

This project will create a robust, flexible, and secure network infrastructure across multiple environments, following **best practices** for **network segmentation** and **security** in AWS.

Let me know if you'd like further customization or specific examples for any of the additional networking features!

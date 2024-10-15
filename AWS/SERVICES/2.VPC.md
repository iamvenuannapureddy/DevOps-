<H1>AWS VPC</H1>

### What is AWS VPC?

**Amazon Virtual Private Cloud (VPC)** is a service that lets you create a logically isolated section of the AWS cloud where you can launch AWS resources, such as EC2 instances, in a virtual network that you define. It gives you complete control over your networking environment, including selecting your own IP address range, creating subnets, configuring route tables, network gateways, and security settings.

#### Key Features of AWS VPC:
1. **Subnets**: VPCs are divided into subnets, which allow you to isolate and control different portions of your network. You can create both **public subnets** (accessible from the internet) and **private subnets** (isolated from direct internet access).

2. **Route Tables**: Route tables define the traffic routing rules within the VPC. They control where network traffic is directed based on destination IP addresses.

3. **Internet Gateway**: This is a component that allows communication between the resources within your VPC and the internet. It is typically attached to public subnets.

4. **NAT Gateway**: For instances in private subnets that need outbound internet access (e.g., for downloading updates), you can use a NAT (Network Address Translation) Gateway.

5. **Security Groups**: Virtual firewalls attached to instances that control inbound and outbound traffic based on rules.

6. **Network ACLs (Access Control Lists)**: Optional layer of security for your subnets that acts as a firewall at the subnet level.

7. **Elastic IP**: A static, public IP address that you can allocate and associate with EC2 instances or other resources in your VPC for consistent access.

8. **VPC Peering**: Allows you to connect two VPCs privately using AWS’s network infrastructure, even if they are in different AWS regions.

9. **VPN and Direct Connect**: VPC can connect to your on-premises data centers via VPN or AWS Direct Connect for hybrid cloud configurations.

10. **Endpoints**: VPC Endpoints allow you to privately connect your VPC to supported AWS services like S3 and DynamoDB without using the public internet.

### How to Create and Use AWS VPC

#### Step-by-Step Process

Let’s walk through creating a VPC, subnets, internet gateway, route tables, and security groups.

---

### 1. **Create a VPC**

You can create a VPC using the AWS Management Console, AWS CLI, or Terraform. Here’s an example using the AWS Console:

1. **Go to the VPC Dashboard**:
   - Sign in to the AWS Management Console, and navigate to **VPC** under Networking & Content Delivery.

2. **Create a VPC**:
   - Click **Create VPC**.
   - Give your VPC a name.
   - Choose an IPv4 CIDR block (e.g., `10.0.0.0/16`), which defines the IP address range for your VPC. The CIDR block should be large enough to accommodate all the subnets you plan to create.
   - Optionally, enable IPv6 (if needed for your application).
   - Choose **Tenancy** (default or dedicated) based on your requirements (most often, you would go with default tenancy).
   - Click **Create**.

---

### 2. **Create Subnets**

Subnets segment your VPC into smaller, more manageable sections. You typically create at least one **public subnet** (for resources like web servers that need internet access) and one **private subnet** (for resources like databases that should be isolated from the internet).

1. **Go to Subnets**:
   - In the VPC Dashboard, click **Subnets**.
   - Click **Create Subnet**.

2. **Define Subnets**:
   - Select the VPC you created.
   - Define the **Subnet Name** (e.g., `Public-Subnet` or `Private-Subnet`).
   - Choose an **Availability Zone** (e.g., `us-east-1a`).
   - Assign an **IPv4 CIDR block** that fits within the VPC CIDR range (e.g., `10.0.1.0/24` for the public subnet and `10.0.2.0/24` for the private subnet).

3. **Create a Subnet** for each section (Public, Private).
   
---

### 3. **Create an Internet Gateway**

The Internet Gateway allows your VPC to communicate with the internet. You attach it to the VPC and associate it with public subnets.

1. **Go to Internet Gateways**:
   - In the VPC Dashboard, click **Internet Gateways**.
   - Click **Create Internet Gateway**.
   - Give it a name and click **Create**.
   
2. **Attach Internet Gateway to VPC**:
   - After creation, select the gateway and click **Actions > Attach to VPC**.
   - Choose the VPC you created.

---

### 4. **Configure Route Tables**

Each subnet in your VPC needs to have a route table associated with it. Public subnets will have a route to the internet via the internet gateway, while private subnets will have internal-only traffic.

1. **Create a Route Table**:
   - Go to **Route Tables** in the VPC Dashboard and click **Create Route Table**.
   - Name it (e.g., `Public-Route-Table`) and associate it with your VPC.

2. **Add Routes**:
   - Select your new route table and click the **Routes** tab.
   - Click **Edit Routes**, then add the following route:
     - **Destination**: `0.0.0.0/0` (for all internet traffic)
     - **Target**: Select the **Internet Gateway** you created earlier.

3. **Associate Route Table with Subnets**:
   - Go to the **Subnet Associations** tab.
   - Click **Edit Subnet Associations** and associate the route table with your public subnets.

---

### 5. **Create a NAT Gateway (Optional for Private Subnet)**

A NAT Gateway allows instances in private subnets to access the internet (for updates, patches, etc.) without exposing them to incoming traffic from the internet.

1. **Go to NAT Gateways**:
   - In the VPC Dashboard, click **NAT Gateways**.
   - Click **Create NAT Gateway**.

2. **Choose the Public Subnet**:
   - Choose a **public subnet** for the NAT Gateway and associate it with an **Elastic IP Address**.

3. **Update the Route Table for Private Subnet**:
   - Go to your **private subnet's route table**.
   - Add a route that directs all internet-bound traffic (`0.0.0.0/0`) to the **NAT Gateway**.

---

### 6. **Security Groups**

Security Groups control inbound and outbound traffic to EC2 instances. You can define rules to allow/deny traffic.

1. **Go to Security Groups**:
   - In the EC2 or VPC Dashboard, click **Security Groups**.
   - Click **Create Security Group**.

2. **Define Rules**:
   - Name the security group (e.g., `Web-Security-Group`).
   - Choose the VPC it will apply to.
   - Add **inbound rules** (e.g., allowing HTTP traffic on port 80 from `0.0.0.0/0` or HTTPS traffic on port 443).
   - Add **outbound rules** (by default, all outbound traffic is allowed).

3. **Attach Security Group to EC2 Instances**:
   - When launching an EC2 instance, you can select this security group to control access.

---

### 7. **Test Your VPC**

After setting up the VPC, subnets, internet gateway, and security groups:
1. **Launch an EC2 Instance** in your **public subnet**.
   - Attach the appropriate security group to allow SSH (port 22) or HTTP/HTTPS traffic.
2. **Access the instance** via the internet using the public IP address.
3. **Launch an EC2 instance in the private subnet** (optionally using a NAT Gateway for outbound traffic).

---

### Terraform Example to Create a VPC

Here’s an example of how you can create a VPC using **Terraform**:

```hcl
provider "aws" {
  region = "us-east-1"
}

# Create a VPC
resource "aws_vpc" "my_vpc" {
  cidr_block = "10.0.0.0/16"
  enable_dns_support   = true
  enable_dns_hostnames = true

  tags = {
    Name = "my-vpc"
  }
}

# Create a public subnet
resource "aws_subnet" "public_subnet" {
  vpc_id            = aws_vpc.my_vpc.id
  cidr_block        = "10.0.1.0/24"
  map_public_ip_on_launch = true

  tags = {
    Name = "public-subnet"
  }
}

# Create a private subnet
resource "aws_subnet" "private_subnet" {
  vpc_id            = aws_vpc.my_vpc.id
  cidr_block        = "10.0.2.0/24"

  tags = {
    Name = "private-subnet"
  }
}

# Create an Internet Gateway
resource "aws_internet_gateway" "my_igw" {
  vpc_id = aws_vpc.my_vpc.id

  tags = {
    Name = "my-igw"
  }
}

# Create a route table for the public subnet
resource "aws_route_table" "public_route_table" {
  vpc_id

 = aws_vpc.my_vpc.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.my_igw.id
  }

  tags = {
    Name = "public-route-table"
  }
}

# Associate the route table with the public subnet
resource "aws_route_table_association" "public_route_assoc" {
  subnet_id      = aws_subnet.public_subnet.id
  route_table_id = aws_route_table.public_route_table.id
}

# Create a NAT Gateway for private subnets (optional)
resource "aws_eip" "nat_eip" {
  vpc = true
}

resource "aws_nat_gateway" "my_nat_gateway" {
  allocation_id = aws_eip.nat_eip.id
  subnet_id     = aws_subnet.public_subnet.id
}

# Route for private subnet using NAT Gateway
resource "aws_route_table" "private_route_table" {
  vpc_id = aws_vpc.my_vpc.id

  route {
    cidr_block     = "0.0.0.0/0"
    nat_gateway_id = aws_nat_gateway.my_nat_gateway.id
  }

  tags = {
    Name = "private-route-table"
  }
}

resource "aws_route_table_association" "private_route_assoc" {
  subnet_id      = aws_subnet.private_subnet.id
  route_table_id = aws_route_table.private_route_table.id
}
```

This Terraform configuration will:
- Create a VPC with both public and private subnets.
- Attach an Internet Gateway to the public subnet.
- Optionally, create a NAT Gateway for the private subnet.

You can expand upon this by adding security groups, EC2 instances, and other components based on your needs.

---

### Use Cases of AWS VPC

- **Hosting Web Applications**: Public-facing web servers in a public subnet and databases in a private subnet.
- **Isolated Environments**: Create secure, isolated environments for sensitive workloads.
- **Hybrid Cloud**: Use VPN or Direct Connect to integrate on-premises networks with AWS VPC for hybrid cloud architectures.
- **Microservices**: Launch microservices across multiple subnets, using security groups and network ACLs for security.

Would you like to explore any specific VPC components in more detail, or go deeper into any other networking features of AWS VPC?

<H1> AWS EC2</H1>

### What is AWS EC2?

**Amazon EC2 (Elastic Compute Cloud)** is a web service that provides resizable compute capacity in the cloud. It allows you to launch virtual servers (known as EC2 instances) to run applications, host websites, process data, and more. EC2 instances can be scaled up or down depending on your application needs, offering flexibility and cost-efficiency.

### Key Features of EC2:
1. **Instance Types**: EC2 offers a variety of instance types optimized for different use cases (compute, memory, storage, GPU, etc.), such as general-purpose, compute-optimized, memory-optimized, and storage-optimized instances.
   
2. **Amazon Machine Images (AMIs)**: AMIs are pre-configured templates that provide the information required to launch an instance. They include the operating system, application server, and applications.
   
3. **Elastic Block Store (EBS)**: EBS provides persistent block storage for your EC2 instances. It can be attached, detached, and resized dynamically.

4. **Elastic IPs**: A static, public IP address that can be associated with your EC2 instance, allowing you to have a persistent IP address that can be reassigned.

5. **Auto Scaling**: Automatically adjusts the number of EC2 instances in response to traffic patterns.

6. **Elastic Load Balancing (ELB)**: Distributes incoming application traffic across multiple EC2 instances for fault tolerance.

7. **Security Groups**: A virtual firewall that controls inbound and outbound traffic to instances.

8. **Key Pairs**: EC2 uses SSH key pairs to authenticate users trying to connect to the instance.

9. **Instance Store vs. EBS-backed instances**: EC2 provides two types of storage: ephemeral instance storage (fast but non-persistent) and EBS volumes (persistent and reliable).

10. **Placement Groups**: Logical grouping of instances to influence the placement of instances on physical hardware, enabling higher performance or availability.

11. **Spot, Reserved, and On-Demand Instances**:
   - **On-Demand Instances**: Pay-as-you-go with no upfront cost.
   - **Reserved Instances**: Commit to a specific instance for 1 or 3 years in exchange for a discounted rate.
   - **Spot Instances**: Use spare AWS capacity at a discounted rate, with the possibility of interruption.

---

### How to Create and Use an EC2 Instance

#### Step-by-Step Guide to Creating an EC2 Instance

1. **Go to the EC2 Dashboard**:
   - Log in to the **AWS Management Console** and navigate to **EC2** under the "Compute" section.

---

### Step 1: Launch an EC2 Instance

1. **Click "Launch Instance"**:
   - Click the **Launch Instance** button on the EC2 Dashboard.

2. **Choose an Amazon Machine Image (AMI)**:
   - Choose an AMI that contains the software configuration you need.
   - Options include:
     - **Amazon Linux 2**: A popular choice optimized for EC2.
     - **Ubuntu**, **RedHat**, **Windows**, or other Linux distributions.

3. **Choose an Instance Type**:
   - Select the instance type based on your compute, memory, and storage requirements.
   - Common types include:
     - **t2.micro** (Free tier eligible): Suitable for low traffic or testing environments.
     - **m5.large**: General-purpose instance with more CPU and memory for medium workloads.
     - **r5.large**: Memory-optimized instance for memory-heavy applications.

4. **Configure Instance Details**:
   - Set up instance details such as:
     - Number of instances.
     - Network: Choose your **VPC** (Virtual Private Cloud) and **subnet**.
     - Auto-assign Public IP: Enable if you want the instance to be publicly accessible.
     - IAM Role: Optionally, attach an IAM role to allow the instance to access AWS services securely.
     - Shutdown Behavior: Choose whether to stop or terminate the instance when it's shut down.
     - Enable termination protection to prevent accidental termination.

5. **Add Storage**:
   - By default, an **EBS volume** is attached to the instance.
   - You can add more volumes, or modify the existing one (e.g., size or type).
     - **General Purpose SSD (gp3)**: Suitable for general workloads.
     - **Provisioned IOPS SSD (io2)**: Ideal for high-performance applications like databases.

6. **Configure Security Group**:
   - Create or select a **Security Group**, which acts as a virtual firewall for your instance.
     - Add rules for **inbound traffic** (e.g., allow SSH traffic on port 22 if you want to connect via SSH).
     - For web servers, allow HTTP (port 80) or HTTPS (port 443) traffic.

7. **Key Pair**:
   - Create a **key pair** (or use an existing one) for SSH access to the instance.
   - Download the **private key file (.pem)** and keep it secure. You will need it to access your instance.

8. **Review and Launch**:
   - Review your settings and click **Launch**.
   - Choose the **key pair** (for SSH access) and acknowledge that you have access to the private key.

---

### Step 2: Access Your EC2 Instance

Once the instance is launched, it will take a few minutes to be in the "running" state.

1. **Find the Public IP**:
   - Go to the **EC2 Dashboard** and view your instances.
   - Find the **public IP address** or **DNS** of your EC2 instance.

2. **SSH into the EC2 Instance** (Linux/Ubuntu/Mac):
   - Open a terminal and run the following command:
   
     ```bash
     ssh -i /path/to/your-key.pem ec2-user@<public-ip>
     ```
   - Replace `/path/to/your-key.pem` with the path to your downloaded key file and `<public-ip>` with the public IP of the instance.

   **Example**:
   ```bash
   ssh -i ~/Downloads/my-key.pem ec2-user@3.128.101.104
   ```

3. **Connect using PuTTY** (for Windows):
   - Convert the `.pem` file to a `.ppk` file using **PuTTYgen**.
   - Use **PuTTY** to connect by entering the public IP and specifying the `.ppk` file for authentication.

---

### Step 3: Manage EC2 Instances

1. **Start/Stop/Terminate Instances**:
   - You can start, stop, or terminate an instance from the EC2 dashboard.
   - **Stopping** retains the EBS volumes and can restart the instance later.
   - **Terminating** permanently deletes the instance and associated data (unless you use EBS volumes with "Delete on termination" disabled).

2. **Monitor Performance**:
   - Use **CloudWatch** to monitor instance performance, including CPU utilization, memory usage, disk I/O, and network traffic.

3. **Scaling**:
   - You can manually resize an EC2 instance by stopping it and changing the **instance type**.
   - For auto-scaling, you can use **Auto Scaling Groups** to automatically add or remove instances based on demand.

---

### Step 4: Use EC2 with Other AWS Services

1. **Elastic Load Balancing (ELB)**:
   - Distribute incoming traffic across multiple EC2 instances to ensure high availability and fault tolerance.

2. **Auto Scaling**:
   - Automatically increase or decrease the number of instances based on specified conditions, such as CPU usage or request count.

3. **Amazon RDS or Aurora**:
   - Use Amazon RDS or Aurora as your managed relational database solution to host data for your application.

4. **S3 for Object Storage**:
   - Store static files such as images or backups on **Amazon S3**, while EC2 instances handle application processing.

5. **Elastic IP (EIP)**:
   - Allocate an **Elastic IP** to ensure that your instance retains a public IP address, even if it is stopped and restarted.

---

### Terraform Example to Launch an EC2 Instance

Here’s how you can create an EC2 instance using **Terraform**:

```hcl
provider "aws" {
  region = "us-east-1"
}

# Define the key pair (use your actual key name)
resource "aws_key_pair" "my_key" {
  key_name   = "my-key"
  public_key = file("~/.ssh/id_rsa.pub") # Replace with your public key path
}

# Create a Security Group allowing SSH and HTTP
resource "aws_security_group" "web_sg" {
  name        = "web-sg"
  description = "Allow SSH and HTTP traffic"

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"] # Allow SSH from anywhere
  }

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"] # Allow HTTP traffic
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

# Create an EC2 instance
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"  # Example AMI for Amazon Linux 2
  instance_type =

 "t2.micro"

  key_name      = aws_key_pair.my_key.key_name
  security_groups = [aws_security_group.web_sg.name]

  tags = {
    Name = "my-web-server"
  }
}
```

This Terraform code does the following:
- Creates an EC2 instance using a specific AMI and instance type.
- Configures security groups to allow SSH and HTTP traffic.
- Associates a key pair for SSH access.

---

### EC2 Use Cases

1. **Web Hosting**: Hosting dynamic websites or web applications on scalable EC2 instances.
2. **Data Processing**: Running data processing or machine learning jobs on EC2 instances.
3. **Development and Testing**: Deploying development and testing environments on-demand.
4. **High-Performance Computing**: Use compute-optimized instances for workloads like scientific simulations or rendering.
5. **Gaming Servers**: Hosting online multiplayer game servers using EC2 instances.

---

### Conclusion

AWS EC2 is a highly flexible and scalable cloud service that allows you to run virtual servers to meet a wide range of use cases, from simple web hosting to complex, resource-intensive applications. With EC2, you have fine-grained control over your compute resources and can seamlessly integrate them with other AWS services.

Would you like to explore specific EC2 features in more detail or see more advanced configurations (e.g., using Auto Scaling or Spot instances)?

<h1>EC2-ASG</h1>

When working with **EC2 instances** and **Auto Scaling Groups (ASGs)** using Terraform, the two are tightly connected because an **Auto Scaling Group** manages the lifecycle of EC2 instances. While EC2 instances provide compute resources, Auto Scaling ensures that the right number of instances are running to handle the demand for your application. Let’s break down the relationship and configurations you need to know to work with **EC2** and **Auto Scaling** together.

### 1. **EC2 Instance Creation**

When manually creating an EC2 instance, you typically specify its configuration, such as the Amazon Machine Image (AMI), instance type, and key pair. In the context of an Auto Scaling Group (ASG), however, you don't create EC2 instances directly. Instead, the ASG launches, terminates, and manages the EC2 instances according to your scaling policies.

#### EC2 Instance Key Specifications:
- **AMI ID**: The Amazon Machine Image ID, which defines the OS and software configuration.
- **Instance Type**: The type of EC2 instance (e.g., `t2.micro`, `m5.large`).
- **Key Pair**: SSH key to access the instance.
- **Security Groups**: Firewall rules controlling traffic to the instance.
- **IAM Role**: Optional role for the instance to access AWS services.
- **User Data**: Optional script that runs on instance launch for configuration (e.g., installing packages).

#### Example (Standalone EC2 Instance Creation):
```hcl
resource "aws_instance" "my_ec2_instance" {
  ami           = "ami-12345678"
  instance_type = "t2.micro"
  key_name      = "my-key"
  subnet_id     = aws_subnet.public_subnet.id

  security_groups = [aws_security_group.my_sg.id]

  user_data = <<-EOF
    #!/bin/bash
    yum install -y httpd
    systemctl start httpd
  EOF

  tags = {
    Name = "MyEC2Instance"
  }
}
```

In this case, the EC2 instance is manually provisioned. But with **Auto Scaling**, you automate this process by defining an ASG that scales based on demand.

---

### 2. **Auto Scaling Group (ASG)**

An **Auto Scaling Group** manages a fleet of EC2 instances, ensuring that the correct number of instances is running. You don't create instances directly in an ASG; instead, you define the **Launch Template** (or **Launch Configuration**) that specifies how the EC2 instances should be launched.

#### ASG Key Specifications:
- **Launch Template** or **Launch Configuration**: Defines the EC2 instance configuration.
- **Desired Capacity**: The number of instances to maintain.
- **Min Size** and **Max Size**: The minimum and maximum number of instances in the ASG.
- **VPC Subnets**: Specifies where the instances are launched.
- **Scaling Policies**: Automatically scales in and out based on demand.

#### Example (EC2 with Auto Scaling Group):

```hcl
# Launch Template for EC2 Instances
resource "aws_launch_template" "my_launch_template" {
  name          = "my-launch-template"
  image_id      = "ami-12345678"
  instance_type = "t2.micro"

  key_name = "my-key"

  network_interfaces {
    associate_public_ip_address = true
    security_groups             = [aws_security_group.my_sg.id]
  }

  user_data = <<-EOF
    #!/bin/bash
    yum install -y httpd
    systemctl start httpd
  EOF
}

# Auto Scaling Group
resource "aws_autoscaling_group" "my_asg" {
  launch_template {
    id      = aws_launch_template.my_launch_template.id
    version = "$Latest"
  }

  vpc_zone_identifier = [aws_subnet.public_subnet.id]
  min_size            = 2
  max_size            = 5
  desired_capacity    = 3

  health_check_type         = "EC2"
  health_check_grace_period = 300  # Seconds to wait before performing health check

  tag {
    key                 = "Name"
    value               = "MyASGInstance"
    propagate_at_launch = true
  }
}
```

---

### 3. **Scaling Policies for the ASG**
Scaling policies automatically adjust the number of EC2 instances in your ASG. You can define policies that trigger scaling events based on CPU usage, memory, or any other CloudWatch metrics.

#### Example of Scaling Policies:

```hcl
# Scale-out policy (increase capacity by 1 instance)
resource "aws_autoscaling_policy" "scale_out" {
  name                   = "scale-out-policy"
  scaling_adjustment      = 1
  adjustment_type         = "ChangeInCapacity"
  cooldown               = 300
  autoscaling_group_name  = aws_autoscaling_group.my_asg.name
}

# Scale-in policy (decrease capacity by 1 instance)
resource "aws_autoscaling_policy" "scale_in" {
  name                   = "scale-in-policy"
  scaling_adjustment      = -1
  adjustment_type         = "ChangeInCapacity"
  cooldown               = 300
  autoscaling_group_name  = aws_autoscaling_group.my_asg.name
}
```

---

### 4. **Instance Lifecycle in an Auto Scaling Group**
The ASG handles the lifecycle of EC2 instances:
1. **Launching**: The ASG launches new EC2 instances when the desired capacity or scaling policies require it.
2. **Terminating**: When demand decreases, the ASG terminates instances based on the scaling policies.
3. **Replacements**: If an instance becomes unhealthy (failing health checks), the ASG terminates it and replaces it with a new one.

#### Example of Lifecycle Hooks (Optional):

You can add lifecycle hooks to perform actions when an instance is about to be terminated or launched. This could be used to drain connections or store logs.

```hcl
resource "aws_autoscaling_lifecycle_hook" "terminate_hook" {
  autoscaling_group_name = aws_autoscaling_group.my_asg.name
  name                   = "terminate-hook"
  lifecycle_transition   = "autoscaling:EC2_INSTANCE_TERMINATING"
  default_result         = "CONTINUE"
  heartbeat_timeout      = 300

  notification_target_arn = aws_sns_topic.lifecycle_notifications.arn
  role_arn               = aws_iam_role.my_role.arn
}
```

---

### Key Considerations:
1. **AMI (Amazon Machine Image)**: The AMI defines the operating system and applications running on the EC2 instances.
2. **Instance Type**: Choose the right instance type based on the compute and memory requirements of your application.
3. **Scaling Policies**: Ensure that your scaling policies are aligned with your application's demand patterns to avoid over/under-provisioning.
4. **Health Checks**: Use health checks to automatically replace unhealthy EC2 instances.
5. **High Availability**: Distribute instances across multiple Availability Zones (AZs) to ensure fault tolerance.

---

### Summary: EC2 + Auto Scaling Group
- **EC2 Instances**: Provides the compute resources.
- **Auto Scaling Group (ASG)**: Automatically manages the lifecycle of EC2 instances by launching, terminating, and scaling based on predefined rules.
- **Launch Template**: Defines how the EC2 instances are configured in the ASG.
- **Scaling Policies**: Dynamically adjust the number of instances based on demand or custom metrics.

Together, **EC2 instances** and **Auto Scaling Groups** ensure that your infrastructure is scalable, resilient, and cost-effective.

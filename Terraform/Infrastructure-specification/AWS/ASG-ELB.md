<h1>ASG-ELB</h1>
When working with **Auto Scaling** and **Load Balancers** in AWS using Terraform, they help provide a scalable and resilient infrastructure by automatically adjusting the number of EC2 instances and distributing incoming traffic across these instances. Below are the key components and configurations you need to know when creating and managing Auto Scaling Groups (ASGs) and Load Balancers (ALBs or NLBs) with Terraform.

### 1. **Auto Scaling Group (ASG)**
An **Auto Scaling Group (ASG)** automatically manages a fleet of EC2 instances by scaling in and out based on policies or schedules.

#### Key Specifications:
- **Launch Template** or **Launch Configuration**: Defines the AMI, instance type, key pairs, etc., that ASG will use to launch instances.
- **Desired Capacity**: The number of instances that ASG maintains.
- **Min Size** and **Max Size**: The minimum and maximum number of instances the ASG should have.
- **VPC Subnets**: Specifies the subnets where the instances will be launched.
- **Scaling Policies**: Defines how the ASG scales in or out based on CPU utilization or other CloudWatch metrics.

#### Example:

```hcl
# Launch Template
resource "aws_launch_template" "app_launch_template" {
  name = "my-app-launch-template"

  image_id      = "ami-12345678"
  instance_type = "t2.micro"

  key_name = "my-key"

  network_interfaces {
    associate_public_ip_address = true
    security_groups             = [aws_security_group.instance.id]
  }

  user_data = <<-EOF
    #!/bin/bash
    yum install -y httpd
    systemctl start httpd
  EOF
}

# Auto Scaling Group
resource "aws_autoscaling_group" "app_asg" {
  launch_template {
    id      = aws_launch_template.app_launch_template.id
    version = "$Latest"
  }

  vpc_zone_identifier = [aws_subnet.public_subnet.id]
  min_size            = 2
  max_size            = 5
  desired_capacity    = 3

  target_group_arns = [aws_lb_target_group.app_tg.arn]

  tag {
    key                 = "Name"
    value               = "my-asg-instance"
    propagate_at_launch = true
  }
}

# Auto Scaling Policies (based on CPU utilization)
resource "aws_autoscaling_policy" "scale_out" {
  name                   = "scale-out-policy"
  scaling_adjustment      = 1
  adjustment_type         = "ChangeInCapacity"
  cooldown               = 300
  autoscaling_group_name  = aws_autoscaling_group.app_asg.name

  metric_aggregation_type = "Average"
  estimated_instance_warmup = 180
}
```

---

### 2. **Load Balancer**
AWS **Load Balancers** distribute incoming application traffic across multiple targets, such as EC2 instances. You can use either an **Application Load Balancer (ALB)** for HTTP/HTTPS traffic or a **Network Load Balancer (NLB)** for TCP traffic.

#### 2.1 **Application Load Balancer (ALB)**
An ALB operates at Layer 7 (HTTP/HTTPS) and provides advanced routing features like path-based routing and host-based routing.

#### Key Specifications:
- **Security Groups**: Controls the inbound traffic to the load balancer.
- **Listeners**: Configures the port and protocol (e.g., HTTP or HTTPS) on which the ALB listens.
- **Target Groups**: Defines the EC2 instances or services that the ALB forwards requests to.
- **Health Checks**: Monitors the health of the targets and removes unhealthy ones from the load balancer.

#### Example:

```hcl
# Security Group for Load Balancer
resource "aws_security_group" "alb_sg" {
  name   = "alb-security-group"
  vpc_id = aws_vpc.main.id

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

# Application Load Balancer (ALB)
resource "aws_lb" "app_lb" {
  name               = "my-app-lb"
  internal           = false
  load_balancer_type = "application"
  security_groups    = [aws_security_group.alb_sg.id]
  subnets            = [aws_subnet.public_subnet.id]

  enable_deletion_protection = false

  tags = {
    Name = "AppLB"
  }
}

# ALB Target Group
resource "aws_lb_target_group" "app_tg" {
  name     = "app-tg"
  port     = 80
  protocol = "HTTP"
  vpc_id   = aws_vpc.main.id

  health_check {
    path                = "/"
    interval            = 30
    timeout             = 5
    healthy_threshold   = 2
    unhealthy_threshold = 2
    matcher             = "200"
  }

  tags = {
    Name = "AppTargetGroup"
  }
}

# ALB Listener
resource "aws_lb_listener" "app_listener" {
  load_balancer_arn = aws_lb.app_lb.arn
  port              = 80
  protocol          = "HTTP"

  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.app_tg.arn
  }
}
```

---

#### 2.2 **Network Load Balancer (NLB)**
A **Network Load Balancer (NLB)** operates at Layer 4 (TCP) and is designed for high-performance traffic, handling millions of requests per second with ultra-low latencies.

- **Key Specifications**:
  - `subnets`: The subnets to associate the NLB with.
  - `listeners`: Define the TCP or TLS ports and protocols.
  - `target_group`: Define which targets (instances, IPs, or Lambda functions) the NLB forwards traffic to.

#### Example:

```hcl
# Network Load Balancer (NLB)
resource "aws_lb" "app_nlb" {
  name               = "my-app-nlb"
  internal           = false
  load_balancer_type = "network"
  subnets            = [aws_subnet.public_subnet.id]

  enable_deletion_protection = false

  tags = {
    Name = "AppNLB"
  }
}

# NLB Target Group
resource "aws_lb_target_group" "nlb_tg" {
  name     = "app-nlb-tg"
  port     = 80
  protocol = "TCP"
  vpc_id   = aws_vpc.main.id

  health_check {
    interval            = 30
    protocol            = "TCP"
    healthy_threshold   = 3
    unhealthy_threshold = 3
  }

  tags = {
    Name = "NLBTargetGroup"
  }
}

# NLB Listener
resource "aws_lb_listener" "nlb_listener" {
  load_balancer_arn = aws_lb.app_nlb.arn
  port              = 80
  protocol          = "TCP"

  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.nlb_tg.arn
  }
}
```

---

### 3. **Linking Auto Scaling Group with Load Balancer**

When you create an Auto Scaling Group (ASG) and a Load Balancer, you need to ensure that the Load Balancer is aware of the instances in the ASG. This is done by linking the **Target Group** of the Load Balancer to the ASG.

#### Example ASG and ALB Integration:
```hcl
# Auto Scaling Group
resource "aws_autoscaling_group" "app_asg" {
  launch_template {
    id      = aws_launch_template.app_launch_template.id
    version = "$Latest"
  }

  vpc_zone_identifier = [aws_subnet.public_subnet.id]
  min_size            = 2
  max_size            = 5
  desired_capacity    = 3

  # Attach Target Group to ASG
  target_group_arns = [aws_lb_target_group.app_tg.arn]

  tag {
    key                 = "Name"
    value               = "my-asg-instance"
    propagate_at_launch = true
  }
}

# Scaling Policies based on CPU utilization or custom metrics
resource "aws_autoscaling_policy" "scale_out" {
  name                   = "scale-out-policy"
  scaling_adjustment      = 1
  adjustment_type         = "ChangeInCapacity"
  cooldown               = 300
  autoscaling_group_name  = aws_autoscaling_group.app_asg.name

  metric_aggregation_type = "Average"
}
```

---

### Key Considerations:
1. **Health Checks**: Ensure that health checks are properly configured for both the ASG and the Load Balancer to automatically replace unhealthy instances.
2. **Scaling Policies**: Set scaling policies based on CloudWatch metrics like CPU utilization, request count, or custom metrics.
3. **Elasticity and High Availability**: Place instances across multiple Availability Zones (AZs) to ensure high availability.
4. **Security**: Ensure that security groups are correctly set for both the Load Balancer and the EC2 instances in the ASG.

By combining Auto Scaling and Load Balancers, you create a dynamic, scalable, and resilient infrastructure

 that can automatically adjust to varying loads.

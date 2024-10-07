<h1>Project-2</h1>

### **Project: Deploying Auto Scaling and Load Balancing Using Terraform**

In this project, you will deploy a highly available web application using an **Auto Scaling Group (ASG)** and an **Application Load Balancer (ALB)**. Terraform will manage the infrastructure, including configuring auto-scaling policies based on **CloudWatch** alarms.

---

### **1. Project Directory Structure**

Here’s a suggested directory structure for organizing your Terraform files and reusable modules:

```bash
aws-auto-scaling-lb/
│
├── modules/
│   ├── ec2-instance/                   # Reusable module for EC2 instances
│   ├── autoscaling-group/              # Module for Auto Scaling Group
│   └── load-balancer/                  # Module for Application Load Balancer (ALB)
│
├── dev/                                # Configuration for dev environment
│   ├── main.tf                         # Dev environment configurations
│   ├── variables.tf                    # Variables specific to dev
│   └── terraform.tfvars                # Environment-specific variables
├── test/                               # Configuration for test environment
│   ├── main.tf                         # Test environment configurations
│   ├── variables.tf                    # Variables specific to test
│   └── terraform.tfvars                # Environment-specific variables
├── prod/                               # Configuration for prod environment
│   ├── main.tf                         # Prod environment configurations
│   ├── variables.tf                    # Variables specific to prod
│   └── terraform.tfvars                # Environment-specific variables
├── provider.tf                         # Provider configuration for AWS
├── outputs.tf                          # Global outputs for the infrastructure
├── variables.tf                        # Global variables
└── terraform.tfvars                    # Global variable values
```

---

### **2. Terraform Module for EC2 Instance**

The EC2 instance module will be reused from the previous project to launch instances in the Auto Scaling Group.

### **3. Terraform Module for Auto Scaling Group (ASG)**

In the `modules/autoscaling-group/` directory, define the Auto Scaling Group with scaling policies and launch configurations.

#### **3.1. `main.tf` (ASG Configuration)**

```hcl
resource "aws_launch_configuration" "app" {
  image_id        = var.ami_id
  instance_type   = var.instance_type
  key_name        = var.key_name
  security_groups = [aws_security_group.app_sg.id]
  
  user_data = file("${path.module}/user_data.sh")
}

resource "aws_autoscaling_group" "app" {
  launch_configuration = aws_launch_configuration.app.id
  vpc_zone_identifier  = var.subnet_ids
  desired_capacity     = var.desired_capacity
  max_size             = var.max_size
  min_size             = var.min_size
  health_check_type    = "EC2"
  health_check_grace_period = 300

  target_group_arns = [aws_lb_target_group.app.arn]

  tag {
    key                 = "Name"
    value               = var.instance_name
    propagate_at_launch = true
  }
}

resource "aws_autoscaling_policy" "scale_up" {
  name                   = "scale_up_policy"
  scaling_adjustment     = 1
  adjustment_type        = "ChangeInCapacity"
  cooldown               = 300
  autoscaling_group_name = aws_autoscaling_group.app.name
}

resource "aws_autoscaling_policy" "scale_down" {
  name                   = "scale_down_policy"
  scaling_adjustment     = -1
  adjustment_type        = "ChangeInCapacity"
  cooldown               = 300
  autoscaling_group_name = aws_autoscaling_group.app.name
}
```

#### **3.2. `variables.tf` (ASG Input Variables)**

```hcl
variable "ami_id" {
  description = "AMI ID for the EC2 instance"
  type        = string
}

variable "instance_type" {
  description = "EC2 instance type"
  type        = string
}

variable "key_name" {
  description = "Key pair for SSH access"
  type        = string
}

variable "subnet_ids" {
  description = "List of subnet IDs for ASG"
  type        = list(string)
}

variable "desired_capacity" {
  description = "Desired number of instances"
  type        = number
  default     = 2
}

variable "max_size" {
  description = "Maximum number of instances in the ASG"
  type        = number
  default     = 5
}

variable "min_size" {
  description = "Minimum number of instances in the ASG"
  type        = number
  default     = 1
}

variable "instance_name" {
  description = "Name tag for the instances"
  type        = string
}
```

#### **3.3. `outputs.tf` (ASG Outputs)**

```hcl
output "asg_name" {
  description = "Name of the Auto Scaling Group"
  value       = aws_autoscaling_group.app.name
}
```

---

### **4. Terraform Module for Application Load Balancer (ALB)**

In the `modules/load-balancer/` directory, define the ALB and its target group to balance the traffic across instances.

#### **4.1. `main.tf` (ALB Configuration)**

```hcl
resource "aws_lb" "app" {
  name               = "app-lb"
  internal           = false
  load_balancer_type = "application"
  security_groups    = [aws_security_group.lb_sg.id]
  subnets            = var.subnet_ids
}

resource "aws_lb_target_group" "app" {
  name     = "app-tg"
  port     = 80
  protocol = "HTTP"
  vpc_id   = var.vpc_id
  health_check {
    path                = "/"
    interval            = 30
    timeout             = 5
    healthy_threshold   = 5
    unhealthy_threshold = 2
  }
}

resource "aws_lb_listener" "app" {
  load_balancer_arn = aws_lb.app.arn
  port              = "80"
  protocol          = "HTTP"
  
  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.app.arn
  }
}
```

#### **4.2. `variables.tf` (ALB Input Variables)**

```hcl
variable "subnet_ids" {
  description = "List of subnet IDs for ALB"
  type        = list(string)
}

variable "vpc_id" {
  description = "VPC ID"
  type        = string
}
```

#### **4.3. `outputs.tf` (ALB Outputs)**

```hcl
output "alb_dns_name" {
  description = "DNS name of the Application Load Balancer"
  value       = aws_lb.app.dns_name
}
```

---

### **5. CloudWatch Alarms for Auto Scaling**

Use CloudWatch alarms to trigger scaling actions based on metrics such as CPU utilization.

#### **5.1. `main.tf` (CloudWatch Alarm)**

```hcl
resource "aws_cloudwatch_metric_alarm" "cpu_high" {
  alarm_name          = "high_cpu"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = 2
  metric_name         = "CPUUtilization"
  namespace           = "AWS/EC2"
  period              = 60
  statistic           = "Average"
  threshold           = 80
  alarm_actions       = [aws_autoscaling_policy.scale_up.arn]
  dimensions = {
    AutoScalingGroupName = aws_autoscaling_group.app.name
  }
}

resource "aws_cloudwatch_metric_alarm" "cpu_low" {
  alarm_name          = "low_cpu"
  comparison_operator = "LessThanThreshold"
  evaluation_periods  = 2
  metric_name         = "CPUUtilization"
  namespace           = "AWS/EC2"
  period              = 60
  statistic           = "Average"
  threshold           = 20
  alarm_actions       = [aws_autoscaling_policy.scale_down.arn]
  dimensions = {
    AutoScalingGroupName = aws_autoscaling_group.app.name
  }
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

3. **Apply the Infrastructure**:

   ```bash
   terraform apply -var-file="dev/terraform.tfvars"
   ```

4. **Scaling Events**:
   Once deployed, your EC2 instances will automatically scale up when the CPU utilization crosses 80% and scale down when it falls below 20%.

---

### **7. Environment-Specific Configurations (dev, test, prod)**

Each environment (dev, test, prod) can have separate instance types, desired capacities, and scaling policies. For example:

- **Development**: Smaller instances and minimal scaling (1–2 instances).
- **Testing**: Moderate-sized instances with slightly more scaling (1–3 instances).
- **Production**: High-performance instances with aggressive scaling (1–5 instances).

You can customize the `terraform.tfvars` file in each environment to fine-tune the configurations.

---

### **8. Conclusion**

By using this Terraform project, you can automate the deployment of a highly available web application with automatic scaling based on load. The Application Load Balancer ensures traffic is evenly distributed, while the Auto Scaling Group maintains the desired instance count based on

 resource usage. The configuration is flexible enough to apply to multiple environments with minimal changes.

Let me know if you need any additional information or further customization!

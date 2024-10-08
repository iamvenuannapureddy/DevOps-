<h1>Project-5</h1>

### **Serverless Application with Lambda using Terraform**

#### **Scenario Overview:**
You're deploying a serverless application that consists of:
- **AWS Lambda** for backend logic.
- **API Gateway** to expose the Lambda functions as HTTP endpoints.
- **DynamoDB** to store persistent data.
- **IAM roles** to control the permissions between the services.

### **Directory Structure**

```
serverless-app/
├── main.tf               # Main Terraform config
├── variables.tf          # Variable definitions
├── outputs.tf            # Output values
├── dev/terraform.tfvars  # Development environment variables
├── prod/terraform.tfvars # Production environment variables
└── modules/
    ├── lambda/
    │   ├── main.tf       # Lambda function resource definitions
    │   └── variables.tf  # Lambda-specific variables
    ├── api-gateway/
    │   ├── main.tf       # API Gateway definitions
    │   └── variables.tf  # API Gateway variables
    └── dynamodb/
        ├── main.tf       # DynamoDB table definitions
        └── variables.tf  # DynamoDB variables
```

---

### **Main Configuration (`main.tf`)**

```hcl
# Provider Configuration
provider "aws" {
  region = var.aws_region
}

# Lambda Module
module "lambda" {
  source         = "./modules/lambda"
  lambda_name    = var.lambda_name
  lambda_runtime = var.lambda_runtime
  handler        = var.lambda_handler
  s3_bucket      = var.s3_bucket
  s3_key         = var.s3_key
  role_arn       = module.iam_lambda_role.arn
}

# API Gateway Module
module "api_gateway" {
  source         = "./modules/api-gateway"
  api_name       = var.api_name
  lambda_arn     = module.lambda.lambda_arn
}

# DynamoDB Module
module "dynamodb" {
  source      = "./modules/dynamodb"
  table_name  = var.dynamodb_table_name
  read_capacity  = var.dynamodb_read_capacity
  write_capacity = var.dynamodb_write_capacity
}

# IAM Role for Lambda
module "iam_lambda_role" {
  source = "./modules/iam"
  role_name = "lambda-execution-role"
  policy_arn = "arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole"
}
```

---

### **Lambda Module (`modules/lambda/main.tf`)**

```hcl
resource "aws_lambda_function" "this" {
  function_name = var.lambda_name
  s3_bucket     = var.s3_bucket
  s3_key        = var.s3_key
  handler       = var.handler
  runtime       = var.lambda_runtime
  role          = var.role_arn
}

output "lambda_arn" {
  value = aws_lambda_function.this.arn
}
```

---

### **API Gateway Module (`modules/api-gateway/main.tf`)**

```hcl
resource "aws_api_gateway_rest_api" "this" {
  name = var.api_name
}

resource "aws_api_gateway_resource" "resource" {
  rest_api_id = aws_api_gateway_rest_api.this.id
  parent_id   = aws_api_gateway_rest_api.this.root_resource_id
  path_part   = "example"
}

resource "aws_api_gateway_method" "method" {
  rest_api_id   = aws_api_gateway_rest_api.this.id
  resource_id   = aws_api_gateway_resource.resource.id
  http_method   = "GET"
  authorization = "NONE"
}

resource "aws_api_gateway_integration" "integration" {
  rest_api_id = aws_api_gateway_rest_api.this.id
  resource_id = aws_api_gateway_resource.resource.id
  http_method = aws_api_gateway_method.method.http_method
  integration_http_method = "POST"
  type        = "AWS_PROXY"
  uri         = "arn:aws:apigateway:${var.region}:lambda:path/2015-03-31/functions/${var.lambda_arn}/invocations"
}

resource "aws_api_gateway_deployment" "deployment" {
  depends_on = [aws_api_gateway_integration.integration]
  rest_api_id = aws_api_gateway_rest_api.this.id
  stage_name  = "dev"
}
```

---

### **DynamoDB Module (`modules/dynamodb/main.tf`)**

```hcl
resource "aws_dynamodb_table" "this" {
  name         = var.table_name
  hash_key     = "id"
  billing_mode = "PROVISIONED"

  attribute {
    name = "id"
    type = "S"
  }

  read_capacity  = var.read_capacity
  write_capacity = var.write_capacity
}
```

---

### **Variables (`variables.tf`)**

```hcl
variable "aws_region" {
  description = "AWS region"
  default     = "us-east-1"
}

# Lambda variables
variable "lambda_name" {}
variable "lambda_runtime" {
  default = "nodejs14.x"
}
variable "lambda_handler" {
  default = "index.handler"
}
variable "s3_bucket" {}
variable "s3_key" {}

# API Gateway variables
variable "api_name" {}

# DynamoDB variables
variable "dynamodb_table_name" {}
variable "dynamodb_read_capacity" {
  default = 5
}
variable "dynamodb_write_capacity" {
  default = 5
}
```

---

### **Environment-Specific Variables (`dev/terraform.tfvars` and `prod/terraform.tfvars`)**

**`dev/terraform.tfvars`**

```hcl
lambda_name = "dev-example-lambda"
s3_bucket = "dev-lambda-deployments"
s3_key = "example-app.zip"
api_name = "dev-api"
dynamodb_table_name = "dev-items-table"
dynamodb_read_capacity = 5
dynamodb_write_capacity = 5
```

**`prod/terraform.tfvars`**

```hcl
lambda_name = "prod-example-lambda"
s3_bucket = "prod-lambda-deployments"
s3_key = "example-app.zip"
api_name = "prod-api"
dynamodb_table_name = "prod-items-table"
dynamodb_read_capacity = 10
dynamodb_write_capacity = 10
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

- **Lambda**: The Lambda function is deployed from an S3 bucket where the application code (e.g., a Node.js app) is stored. You can update the function by uploading a new ZIP file to S3.
- **API Gateway**: API Gateway acts as the HTTP frontend, triggering the Lambda function on API requests.
- **DynamoDB**: The serverless application stores data in DynamoDB, which can be adjusted for different read/write capacities depending on the environment.
- **IAM Roles**: IAM roles are provisioned to give Lambda access to CloudWatch for logging and DynamoDB for data storage.

This setup can be easily modified for both **dev** and **prod** environments using different `terraform.tfvars` files.

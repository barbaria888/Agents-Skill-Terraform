# Production Terraform Engineering Patterns

Standard architecture and layout patterns derived from industry best practices for building scalable, reliable infrastructure with Terraform.

## 1. Isolated Multi-Environment Directory Pattern

Instead of workspaces, isolate your environments (`stage`, `prod`, `global`) by structuring them into dedicated subdirectories. This limits the blast radius of any single deployment.

```
live/
  global/
    iam/
      main.tf
      outputs.tf
    s3-backend/
      main.tf
      outputs.tf
  stage/
    vpc/
      main.tf
      outputs.tf
    services/
      web-cluster/
        main.tf
        outputs.tf
        variables.tf
    data-stores/
      mysql/
        main.tf
        outputs.tf
  prod/
    vpc/
      main.tf
      outputs.tf
    services/
      web-cluster/
        main.tf
        outputs.tf
        variables.tf
    data-stores/
      mysql/
        main.tf
        outputs.tf
```

## 2. Zero-Downtime Deployment Pattern (ASG Rollover)

When updating the software running in an Auto Scaling Group (ASG) via Launch Configurations, use this pattern to perform a rolling blue/green deployment of EC2 instances with zero downtime.

```hcl
resource "aws_launch_configuration" "example" {
  image_id        = var.ami_id
  instance_type   = "t2.micro"
  security_groups = [aws_security_group.instance.id]
  user_data       = var.user_data

  # 1. Ensure launch config is created before the old one is deleted
  lifecycle {
    create_before_destroy = true
  }
}

resource "aws_autoscaling_group" "example" {
  # 2. Dynamic naming forces replacement of the ASG itself when LC changes
  name                 = "${var.cluster_name}-${aws_launch_configuration.example.name}"
  launch_configuration = aws_launch_configuration.example.name
  vpc_zone_identifier  = var.subnet_ids
  
  target_group_arns = [var.target_group_arn]
  health_check_type = "ELB"

  min_size = 2
  max_size = 10

  # 3. Create the new ASG first, shift traffic, then destroy the old one
  lifecycle {
    create_before_destroy = true
  }
}
```

## 3. Remote State Lock Backend Pattern

Always use a locked remote backend for team environments to avoid concurrent applies from corrupting the state file.

```hcl
terraform {
  required_version = ">= 1.5.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }

  backend "s3" {
    bucket         = "my-org-tf-state-bucket"
    key            = "stage/services/web-cluster/terraform.tfstate"
    region         = "us-east-2"
    dynamodb_table = "my-org-tf-state-locks"
    encrypt        = true
  }
}
```

## 4. Composable Modules Pattern

Write modular configurations by splitting modules into small, single-purpose components (e.g. `vpc`, `security-groups`, `compute`) and orchestrating them from the root module using input/output variables.

### A. Root Module Orchestration (`main.tf`):
```hcl
module "vpc" {
  source = "../../modules/networking/vpc"
  cidr   = "10.0.0.0/16"
}

module "security_groups" {
  source = "../../modules/security/sgs"
  vpc_id = module.vpc.vpc_id # Pass output of VPC to SG module
}

module "web_cluster" {
  source     = "../../modules/compute/web-cluster"
  subnet_ids = module.vpc.public_subnets # Pass output of VPC to compute module
  sg_id      = module.security_groups.web_sg_id
}
```
### B. Benefits:
- **Reusability**: The `vpc` module can be used in staging, production, and testing environments unaltered.
- **Maintainability**: You can validate, format, and run unit tests on individual modules independently.

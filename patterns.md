# Production Terraform Engineering Patterns

Standard architecture, security, state, and refactoring patterns derived from industry best practices for building scalable, reliable infrastructure with Terraform.

---

## 1. Isolated Multi-Environment Directory Pattern

Isolate environments (`stage`, `prod`, `global`) and components (`vpc`, `data-stores`, `services`) into dedicated subdirectories. This enforces strict blast-radius isolation and independent IAM access policies.

```
live/
├── global/
│   ├── s3-backend/          # S3 State Bucket & DynamoDB Lock Table
│   └── iam/                 # Cross-account IAM roles
├── stage/
│   ├── vpc/                 # Network layer
│   ├── data-stores/mysql/   # Database layer
│   └── services/web-cluster/# Compute layer
└── prod/
    ├── vpc/
    ├── data-stores/mysql/
    └── services/web-cluster/
```

---

## 2. Zero-Downtime Deployment Pattern (ASG Rollover)

Perform blue/green rolling deployments of Auto Scaling Group EC2 instances using `create_before_destroy` and dynamic naming triggers.

```hcl
resource "aws_launch_template" "example" {
  name_prefix   = "${var.cluster_name}-lt-"
  image_id      = var.ami_id
  instance_type = "t3.micro"

  user_data = base64encode(templatefile("${path.module}/user_data.sh.tpl", {
    server_port = var.server_port
  }))

  lifecycle {
    create_before_destroy = true
  }
}

resource "aws_autoscaling_group" "example" {
  # Dynamic name forces ASG recreation when Launch Template changes
  name                 = "${var.cluster_name}-${aws_launch_template.example.name_latest_version}"
  vpc_zone_identifier  = var.subnet_ids
  target_group_arns    = [var.target_group_arn]
  health_check_type    = "ELB"

  min_size = 2
  max_size = 10

  launch_template {
    id      = aws_launch_template.example.id
    version = "$Latest"
  }

  lifecycle {
    create_before_destroy = true
  }

  instance_refresh {
    strategy = "Rolling"
    preferences {
      min_healthy_percentage = 50
    }
  }
}
```

---

## 3. Remote State Lock Backend Pattern

Use an encrypted S3 bucket with versioning and a DynamoDB table with `LockID` primary key for team state locking.

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
    region         = "us-east-1"
    dynamodb_table = "my-org-tf-state-locks"
    encrypt        = true
  }
}
```

---

## 4. Secrets Injection Pattern (AWS Secrets Manager / Vault)

Never hardcode passwords or API keys in `.tf` files or `.tfvars`. Fetch secrets dynamically at runtime from AWS Secrets Manager.

```hcl
data "aws_secretsmanager_secret" "db" {
  name = "prod/db/credentials"
}

data "aws_secretsmanager_secret_version" "db" {
  secret_id = data.aws_secretsmanager_secret.db.id
}

locals {
  db_creds = jsondecode(data.aws_secretsmanager_secret_version.db.secret_string)
}

resource "aws_db_instance" "example" {
  identifier_prefix   = "prod-db-"
  engine              = "postgres"
  allocated_storage   = 50
  instance_class      = "db.m6g.large"
  username            = local.db_creds.username
  password            = local.db_creds.password
  skip_final_snapshot = true
}
```

---

## 5. Safe Refactoring Pattern (`moved` Blocks)

Rename resources or move them into modules in code without manual `terraform state mv` commands or resource recreation.

```hcl
# Move resource into a module
moved {
  from = aws_security_group.alb
  to   = module.web_cluster.aws_security_group.alb
}

# Refactor count (list) to for_each (map)
moved {
  from = aws_iam_user.users[0]
  to   = aws_iam_user.users["alice"]
}
```

---

## 6. Brownfield Onboarding Pattern (`import` Blocks)

Declaratively bring pre-existing cloud infrastructure into Terraform state management.

```hcl
import {
  to = aws_s3_bucket.legacy
  id = "my-legacy-bucket-name"
}

# Run: terraform plan -generate-config-out=generated.tf
```

---

## 7. Module Contract Guardrail Pattern (`precondition` & `postcondition`)

Enforce business logic constraints on module inputs and resource attributes.

```hcl
resource "aws_db_instance" "example" {
  identifier_prefix = "app-db-"
  engine            = "postgres"
  allocated_storage = var.db_size_gb
  instance_class    = "db.t3.micro"
  username          = "admin"
  password          = var.db_password

  lifecycle {
    precondition {
      condition     = var.db_size_gb >= 20
      error_message = "Database size must be at least 20 GB."
    }

    postcondition {
      condition     = self.storage_encrypted == true
      error_message = "SECURITY VIOLATION: Database storage must be encrypted."
    }
  }
}
```

---

## 8. Zero-Trust OIDC CI/CD Pipeline Pattern (GitHub Actions)

Authenticate CI/CD runners with cloud providers using short-lived OpenID Connect tokens instead of long-lived access keys.

```yaml
name: "Terraform CD"
on:
  push:
    branches: [ "main" ]

permissions:
  id-token: write
  contents: read

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    - uses: aws-actions/configure-aws-credentials@v2
      with:
        role-to-assume: arn:aws:iam::123456789012:role/GitHubActionsTerraformRole
        aws-region: us-east-1
    - uses: hashicorp/setup-terraform@v2
    - run: terraform init
    - run: terraform apply -auto-approve
```

---

## 9. Non-Blocking Drift Detection Pattern (`check` Blocks)

Assert continuous operational health of deployed resources post-apply.

```hcl
check "api_health" {
  data "http" "status" {
    url = "https://${aws_lb.example.dns_name}/health"
  }

  assert {
    condition     = data.http.status.status_code == 200
    error_message = "ALB health check endpoint returned non-200 status code."
  }
}
```

--- 
name: terraform-up-running
description: >
  Infrastructure-as-Code mastery from "Terraform: Up & Running" (3rd ed., O'Reilly 2023)
  by Yevgeniy Brikman. Covers HCL syntax, state management, modules, loops/conditionals,
  production-grade Terraform patterns, testing strategies, and team workflows.
  Opinionated, production-ready guidance distilled into agent-ready reference.

triggers:
  - terraform
  - iac
  - infrastructure as code
  - hcl
  - terraform state
  - terraform modules
  - terraform backend
  - terraform workspace
  - terraform testing
  - provider
  - terragrunt
  - remote state
  - terraform plan apply
  - resource lifecycle

--- 

<img width="1500" height="27" alt="seperator" src="https://github.com/user-attachments/assets/4d6c5f41-bf43-4f0c-8768-60582937ad58" />

# Terraform: Up & Running — Skill Reference

> *"Terraform is a tool for building, changing, and versioning infrastructure safely and efficiently."*  
> — Yevgeniy Brikman, *Terraform: Up & Running*, 3rd Edition (O'Reilly, 2023)

## How to Use This Skill

| Need | Go To |
|---|---|
| Quick command reference | `cheatsheet.md` |
| Terminology | `glossary.md` |
| Reusable patterns | `patterns.md` |
| Deep chapter detail | `chapters/ch0N.md` |

<p align="center">
  <img 
    src="https://capsule-render.vercel.app/api?type=rect&color=0:4285F4,50:34A853,100:FBBC05&height=3&section=header"
    width="100%"
  />
</p>


## Core Mental Model: The Terraform Workflow

```
Write HCL  →  terraform init  →  terraform plan  →  terraform apply  →  terraform destroy
                ↓                      ↓
         Download providers        Preview diff
         Init backend             (NEVER skip this)
```

**Golden Rules:**
1. **Infrastructure is code** — version it, review it, test it
2. **State is sacred** — never edit it manually; store it remotely
3. **Modules are the unit of reuse** — small, composable, versioned
4. **Plan before you apply** — always review the diff

<p align="center">
  <img 
    src="https://capsule-render.vercel.app/api?type=rect&color=0:4285F4,50:34A853,100:FBBC05&height=3&section=header"
    width="100%"
  />
</p>


## Chapter Map

| Ch | Title | Key Concepts |
| --- | ---| ---|
| 1 | Why Terraform | IaC categories, Terraform vs alternatives |
| 2 | Getting Started | Providers, resources, data sources, variables, outputs |
| 3 | Terraform State | Remote backends, state locking, isolation strategies |
| 4 | Modules | Input vars, outputs, versioning, module registry |
| 5 | Tips & Tricks: Loops, If-Stmts, Deployments | `count`, `for_each`, `dynamic`, `for`, conditionals |
| 6 | Production-Grade Infra | Module design, small modules, composition |
| 7 | Working with Multiple Providers | Multi-region, multi-account, aliases |
| 8 | Production-Grade Terraform Modules | Versioning, release process, module composition |
| 9 | Testing Terraform Code | Unit tests, integration tests, e2e, Terratest |
| 10 | Team Workflows | VCS-driven, Atlantis, Terraform Cloud, CDKTF |

<p align="center">
  <img 
    src="https://capsule-render.vercel.app/api?type=rect&color=0:4285F4,50:34A853,100:FBBC05&height=3&section=header"
    width="100%"
  />
</p>


## HCL Quick Reference

### Resource Block
```hcl
resource "<PROVIDER>_<TYPE>" "<NAME>" {
  argument = value

  lifecycle {
    create_before_destroy = true
    prevent_destroy       = true
    ignore_changes        = [tags]
  }
}
```

### Variable Block
```hcl
variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t3.micro"
  validation {
    condition     = contains(["t3.micro", "t3.small"], var.instance_type)
    error_message = "Must be t3.micro or t3.small."
  }
}
```

### Output Block
```hcl
output "instance_ip" {
  description = "Public IP of the instance"
  value       = aws_instance.example.public_ip
  sensitive   = false
}
```

### Data Source
```hcl
data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"]  # Canonical
  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-*-22.04-amd64-server-*"]
  }
}
```

### Local Values
```hcl
locals {
  env    = "prod"
  prefix = "${var.project}-${local.env}"
  common_tags = {
    Project     = var.project
    Environment = local.env
    ManagedBy   = "terraform"
  }
}
```

<p align="center">
  <img 
    src="https://capsule-render.vercel.app/api?type=rect&color=0:4285F4,50:34A853,100:FBBC05&height=3&section=header"
    width="100%"
  />
</p>


## State Management Decision Tree

```
Where to store state?
  Solo developer / learning?     → local (default, but risky)
  Team / production?             → Remote backend (S3+DynamoDB, GCS, Terraform Cloud)
  
How to isolate environments?
  Workspaces?                    → Avoid for prod isolation (same backend, easy to confuse)
  File layout?                   → RECOMMENDED (separate dirs per env)
  Recommended layout:
    live/
      prod/
        vpc/       ← separate state file
        services/
          web-cluster/
        data-stores/
          mysql/
      staging/
        ...
      global/
        iam/
        s3/
```

<p align="center">
  <img 
    src="https://capsule-render.vercel.app/api?type=rect&color=0:4285F4,50:34A853,100:FBBC05&height=3&section=header"
    width="100%"
  />
</p>


## Module Design Principles (from Ch 4 & 6)

1. **Small modules** — one resource group, one responsibility
2. **Composable** — modules call modules (but avoid deep nesting)
3. **Versioned** — use `?ref=v1.2.0` in source URLs
4. **Explicit inputs/outputs** — no globals, no implicit dependencies

```hcl
# Calling a versioned module
module "web_cluster" {
  source  = "github.com/org/terraform-modules//services/web-cluster?ref=v1.2.3"
  
  cluster_name  = "prod-web"
  instance_type = "t3.small"
  min_size      = 2
  max_size      = 10
}

output "alb_dns_name" {
  value = module.web_cluster.alb_dns_name
}
```

<p align="center">
  <img 
    src="https://capsule-render.vercel.app/api?type=rect&color=0:4285F4,50:34A853,100:FBBC05&height=3&section=header"
    width="100%"
  />
</p>


## Loops & Conditionals Quick Reference

| Feature | Syntax | Use When |
| --- | ---| ---|
|
| `count` | `count = var.enable ? 1 : 0` | Simple enable/disable of a resource |
| `for_each` (map) | `for_each = var.users` | Creating similar resources from a map/set |
| `for` expression | `[for s in var.list : upper(s)]` | Transforming collections in expressions |
| `dynamic` block | `dynamic "ingress" { for_each = ... }` | Repeating nested blocks |
| Conditional | `condition ? true_val : false_val` | Inline ternary in any expression |

```hcl
# for_each example — preferred over count for maps
resource "aws_iam_user" "example" {
  for_each = toset(var.user_names)
  name     = each.value
}

# dynamic block example
resource "aws_security_group" "example" {
  dynamic "ingress" {
    for_each = var.ingress_rules
    content {
      from_port   = ingress.value.from_port
      to_port     = ingress.value.to_port
      protocol    = ingress.value.protocol
      cidr_blocks = ingress.value.cidr_blocks
    }
  }
}
```

<p align="center">
  <img 
    src="https://capsule-render.vercel.app/api?type=rect&color=0:4285F4,50:34A853,100:FBBC05&height=3&section=header"
    width="100%"
  />
</p>


## Production-Grade Checklist

- [ ] Remote backend with state locking
- [ ] State isolation per environment (separate dirs, not workspaces)
- [ ] All modules versioned and pinned
- [ ] `prevent_destroy = true` on critical resources (DBs, etc.)
- [ ] `create_before_destroy` for zero-downtime replacements
- [ ] Sensitive variables marked `sensitive = true`
- [ ] `terraform validate` and `terraform fmt` in CI
- [ ] Automated testing (Terratest or equivalent)
- [ ] `terraform plan` output saved as artifact in CI
- [ ] `terraform apply` only on approved PR merge

<p align="center">
  <img 
    src="https://capsule-render.vercel.app/api?type=rect&color=0:4285F4,50:34A853,100:FBBC05&height=3&section=header"
    width="100%"
  />
</p>


## Anti-Patterns (Never Do These)

| Anti-Pattern | Why Bad | Fix |
| --- | ---| ---|
| Storing state locally | Lost on disk wipe, no locking, no team sharing | Use S3+DynamoDB or Terraform Cloud |
| Editing state files manually | Corrupts state, breaks everything | `terraform state` subcommands only |
| Hardcoding credentials in HCL | Security breach | Environment variables or IAM roles |
| One giant root module | Blast radius, slow plans, hard to reason about | Small focused modules |
| Using workspaces for env isolation | Easy to `apply` to wrong env | Separate directory trees |
| `terraform apply` without plan review | Surprise destructions | Always `plan` first, save output |
| Skipping `terraform init` after provider changes | Stale providers | Always `init` after `.tf` changes |
| No `required_providers` version constraints | Unpredictable upgrades break infra | Pin provider versions |

<img width="1500" height="27" alt="seperator" src="https://github.com/user-attachments/assets/4d6c5f41-bf43-4f0c-8768-60582937ad58" />

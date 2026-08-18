# Production Terraform Code Generation Rules for AI Agents

When tasked with creating, modifying, or reviewing Terraform HCL code, any AI coding agent **MUST** enforce the following 10 rule categories to produce reliable, security-hardened, team-ready infrastructure.

---

## 1. Directory Structure & State Isolation Rules

1. **Never Use Workspaces for Production Isolation**: Use isolated subdirectories (`stage/`, `prod/`, `global/`).
2. **One Backend Per Component Directory**: Every root environment folder MUST define its own `backend "s3"` key path.
3. **Minimize Blast Radius**: Separate networking (`vpc`), databases (`data-stores`), and compute (`services`) into distinct directories with independent state files.

```
live/
├── stage/
│   ├── vpc/                 # State key: stage/vpc/terraform.tfstate
│   ├── data-stores/mysql/   # State key: stage/data-stores/mysql/terraform.tfstate
│   └── services/web-cluster/# State key: stage/services/web-cluster/terraform.tfstate
```

---

## 2. Provider & Lock File Rules

1. **Always Pin Provider Versions**: Use pessimistic constraint `~>` inside `required_providers`.
2. **Commit `.terraform.lock.hcl`**: Always commit the lock file to Git version control.
3. **Declare `configuration_aliases`**: If a child module uses provider aliases, explicitly list them in `required_providers`.

```hcl
terraform {
  required_version = ">= 1.5.0"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}
```

---

## 3. Secrets Security Rules

1. **Zero Plain-Text Secrets**: Never hardcode passwords, private keys, or API tokens in `.tf` files or `.tfvars`.
2. **Use Centralized Secret Stores**: Query AWS Secrets Manager or Vault dynamically using `data` blocks.
3. **Mark Sensitive Attributes**: Always append `sensitive = true` to sensitive variables and outputs.
4. **Encrypt State at Rest**: Always configure `encrypt = true` and S3 bucket SSE-KMS for remote state backends.

```hcl
variable "db_password" {
  type      = string
  sensitive = true
}
```

---

## 4. Resource Iteration Rules

1. **Prefer `for_each` over `count`**: Use `for_each` with maps or sets for resource iteration to prevent index-shift destruction.
2. **Reserve `count` for Toggles**: Use `count` ONLY for boolean enable/disable flags (`count = var.enable ? 1 : 0`).
3. **Use `dynamic` Blocks for Inner Blocks**: Use `dynamic` blocks when repeating nested configuration blocks (like `ingress` or `subnet`).

```hcl
resource "aws_iam_user" "example" {
  for_each = toset(var.user_names)
  name     = each.value
}
```

---

## 5. Module Design Rules

1. **Use Standalone Security Group Rules**: Use `aws_security_group_rule` resources instead of inline `ingress {}` blocks in reusable modules.
2. **Use `${path.module}` for File References**: Always qualify local template or script paths using `${path.module}`.
3. **Keep Modules Focused**: One module = one logical component (e.g. `web-cluster`, `vpc`). Avoid giant multi-purpose modules.
4. **Pin Remote Module Versions**: Always append `?ref=vX.Y.Z` to remote Git module sources.

```hcl
# Inside module
user_data = base64encode(templatefile("${path.module}/user_data.sh.tpl", {
  port = var.server_port
}))
```

---

## 6. Declarative Refactoring Rules

1. **Use `moved` Blocks for Renames**: Never suggest manual `terraform state mv`. Add `moved {}` blocks directly in HCL.
2. **Use `import` Blocks for Brownfield Assets**: Add `import {}` blocks and use `terraform plan -generate-config-out` for onboarding existing assets.
3. **Use `removed` Blocks for Deprecation**: Use `removed { lifecycle { destroy = false } }` to untrack assets safely.

```hcl
moved {
  from = aws_instance.web_server
  to   = aws_instance.web_app
}
```

---

## 7. Zero-Downtime & Lifecycle Rules

1. **Pair Launch Templates and ASGs**: Use `aws_launch_template` with `lifecycle { create_before_destroy = true }`.
2. **Dynamic ASG Naming**: Append the launch template version/hash to the ASG name prefix to force replacement on launch template changes.
3. **Set `prevent_destroy` on Stateful Resources**: Attach `lifecycle { prevent_destroy = true }` to databases, S3 state buckets, and KMS keys.

```hcl
resource "aws_autoscaling_group" "example" {
  name_prefix = "${var.cluster_name}-${aws_launch_template.example.name_latest_version}"

  lifecycle {
    create_before_destroy = true
  }
}
```

---

## 8. Variable & Output Contract Rules

1. **Document Everything**: Provide a clear `description` on every single variable and output.
2. **Enforce `validation` Blocks**: Add `validation {}` blocks to input variables to check boundaries and formats.
3. **Use `precondition` & `postcondition`**: Enforce architectural guardrails directly inside resource `lifecycle {}` blocks.

```hcl
variable "environment" {
  type        = string
  description = "Target deployment environment (stage or prod)"

  validation {
    condition     = contains(["stage", "prod"], var.environment)
    error_message = "Environment must be either 'stage' or 'prod'."
  }
}
```

---

## 9. CI/CD & Authentication Rules

1. **Never Store Static Cloud Keys in CI**: Authenticate GitHub Actions / GitLab CI using OpenID Connect (OIDC) AssumeRole.
2. **Apply Saved Plan Artifacts**: Execute `terraform plan -out=tfplan` in PR checks and run `terraform apply tfplan` on merge.
3. **Automate Formatting & Linting**: Run `terraform fmt -check`, `terraform validate`, and `tflint` on every pull request.

---

## 10. Automated Testing Rules

1. **Shift-Left Static Analysis**: Catch syntax and security flaws using `tflint` and `checkov` before deploying.
2. **Use Native `terraform test`**: Write `.tftest.hcl` unit tests with mock providers for fast, zero-cost logic validation.
3. **Always Register `defer Destroy` in Terratest**: Ensure Go integration tests clean up cloud resources even upon failure.

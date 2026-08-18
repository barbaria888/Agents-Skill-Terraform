# Terraform Command Reference & Decision Cheatsheet

A quick-reference guide for standard Terraform operations, troubleshooting, decision trees, and daily development workflows.

---

## 1. Core Workflow Commands

| Command | Action | Key Options / Flags |
|---|---|---|
| **`terraform init`** | Download providers and initialize remote backend configurations. | `-upgrade` (upgrade provider versions)<br>`-reconfigure` (override backend settings)<br>`-backend-config=path` (partial backend) |
| **`terraform validate`** | Statically check the syntax and structure of HCL files. | N/A |
| **`terraform fmt`** | Formats all HCL files inside the directory to canonical style. | `-check` (exit non-zero if unformatted)<br>`-diff` (show formatting changes) |
| **`terraform plan`** | Generate and show execution plan; dry-run mode. | `-out=path` (save plan artifact)<br>`-var-file=path` (load custom vars)<br>`-target=address` (limit plan scope) |
| **`terraform apply`** | Apply changes to reach desired infrastructure state. | `tfplan` (apply saved plan artifact)<br>`-auto-approve` (skip manual prompt)<br>`-replace=address` (force recreation) |
| **`terraform destroy`** | Tear down all infrastructure managed by current directory. | `-auto-approve` (skip validation warning) |
| **`terraform test`** | Execute native HCL unit/integration tests (`.tftest.hcl`). | `-verbose` (show full test output)<br>`-filter=name` (run specific test) |

---

## 2. Decision Tree: Which Loop Construct to Use?

```
Do you need to repeat a resource or a block?
├── Repeat an entire RESOURCE / MODULE?
│   ├── Simple enable/disable boolean toggle?  → `count = var.enable ? 1 : 0`
│   └── Iterating over a collection/map?       → `for_each = var.my_map` (Preferred for state stability)
├── Repeat an INNER BLOCK inside a resource?
│   └── Nested blocks (ingress, subnet, etc)?  → `dynamic "ingress" { for_each = ... }`
└── Transform a collection inside expressions? → `for` expression (`[for s in list : upper(s)]`)
```

---

## 3. Decision Tree: How to Manage Secrets?

```
Where is the secret used?
├── Secret needed in CI/CD cloud authentication?
│   └── Use OpenID Connect (OIDC) AssumeRole (No static keys in CI secrets!)
├── Secret needed inside HCL code execution?
│   ├── Central secret store available (AWS SM / Vault)? → Query via `data` blocks at runtime
│   └── No central store available?                      → Pass via `TF_VAR_<name>` environment variables
└── Secret exported as output?
    └── Mark output with `sensitive = true` (Remembers state files still hold plain text JSON!)
```

---

## 4. Decision Tree: Refactoring Infrastructure State

```
What refactoring action are you taking?
├── Renaming a resource or moving it into a module? → Add a `moved {}` block in HCL
├── Onboarding existing "brownfield" cloud asset?     → Add an `import {}` block + `plan -generate-config-out`
├── Removing resource from TF without destroying it? → Add a `removed {}` block (`destroy = false`)
└── Debugging raw state attributes?                → Use `terraform state list / show / mv / rm`
```

---

## 5. State Management Commands

```bash
# List all resources currently tracked in state
terraform state list

# Show detailed state properties for a specific resource
terraform state show aws_instance.example

# Remove a resource from state tracking (does not destroy live cloud asset)
terraform state rm aws_instance.example

# Rename or move a resource in state tracking manually
terraform state mv aws_instance.old_name aws_instance.new_name
```

---

## 6. Declarative Onboarding & Import

```bash
# Declarative import via HCL (Terraform 1.5+)
# 1. Add import block in HCL: import { to = aws_s3_bucket.foo id = "my-bucket" }
# 2. Run plan to auto-generate code:
terraform plan -generate-config-out=generated.tf

# Imperative legacy import
terraform import aws_instance.imported i-0123456789abcdef0
```

---

## 7. Troubleshooting & Environment Variables

| Variable | Purpose | Example |
|---|---|---|
| **`TF_LOG`** | Controls verbose engine logging level. | `export TF_LOG=DEBUG` (TRACE, DEBUG, INFO, WARN, ERROR) |
| **`TF_LOG_PATH`** | Specifies output file path for log entries. | `export TF_LOG_PATH=./terraform.log` |
| **`TF_VAR_<name>`** | Inject values directly into input variables. | `export TF_VAR_db_password="SecretPassword"` |
| **`TF_DATA_DIR`** | Change location of `.terraform/` cache folder. | `export TF_DATA_DIR="/tmp/tf-cache"` |
| **`TF_CLI_ARGS`** | Append default arguments to CLI commands. | `export TF_CLI_ARGS_apply="-auto-approve"` |

---

## 8. Golden Rules Matrix: Always Do vs. Never Do

| Category | Always Do ✅ | Never Do ❌ |
|---|---|---|
| **State** | Use remote S3 + DynamoDB locking | Store state files in local disk or Git |
| **Secrets** | Use AWS Secrets Manager or `TF_VAR_` | Hardcode passwords in `.tf` or `.tfvars` |
| **Modules** | Pin remote modules to Git tags (`?ref=v1.2.0`) | Reference un-tagged `main` branch (`?ref=main`) |
| **Iteration** | Use `for_each` with maps/sets for resources | Use `count` for lists of named resources |
| **Refactoring** | Use declarative `moved` and `import` blocks | Run manual `terraform state mv` in production |
| **CI/CD** | Apply saved `tfplan` artifacts via OIDC | Run `terraform apply` locally or use static keys |

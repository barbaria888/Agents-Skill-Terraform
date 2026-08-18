# Terraform Command Reference & Cheatsheet

A quick-reference guide for standard Terraform operations, troubleshooting, and daily development workflows.

## 1. Core Workflow Commands

| Command | Action | Key Options / Flags |
|---|---|---|
| **`terraform init`** | Download providers and initialize remote backend configurations. | `-upgrade` (upgrade providers/modules)<br>`-reconfigure` (override backend settings) |
| **`terraform validate`** | Statically check the syntax and structure of files in the folder. | N/A |
| **`terraform fmt`** | Formats all files inside the directory to canonical HCL style. | `-check` (exit non-zero if files are unformatted)<br>`-diff` (show formatting changes) |
| **`terraform plan`** | Generate and show execution plan; dry-run mode. | `-out=path` (save plan artifact)<br>`-var-file=path` (load custom variables)<br>`-target=address` (limit plan to resource) |
| **`terraform apply`** | Apply changes to reach desired state defined in config files. | `tfplan` (apply saved plan)<br>`-auto-approve` (skip manual confirmation)<br>`-replace=address` (force recreation of a resource) |
| **`terraform destroy`** | Tear down all infrastructure managed by the current folder. | `-auto-approve` (skip validation warning) |

## 2. State Management Commands

Manipulate the state file directly without manually editing raw JSON:

```bash
# List all resources currently tracked in the state
terraform state list

# Show detailed state properties for a specific resource
terraform state show aws_instance.example

# Remove a resource from state tracking (does not destroy it)
terraform state rm aws_instance.example

# Rename or move a resource in the state tracking
terraform state mv aws_instance.old_name aws_instance.new_name
```

## 3. Importing Existing Resources

Import existing infrastructure that was created manually into Terraform state control:

```bash
# terraform import <ADDRESS> <ID>
terraform import aws_instance.imported i-0123456789abcdef0
```
*Note: You must define a blank resource block for `aws_instance.imported` in your code before running import.*

## 4. Interactive Console & Output

```bash
# Start an interactive REPL shell to evaluate expressions, functions, or variables
terraform console

# Show outputs defined in configuration
terraform output
terraform output -json
```

## 5. Workspaces

Manage multiple state environments within a single configuration:

```bash
terraform workspace list
terraform workspace new stage
terraform workspace select stage
terraform workspace show
```

## 6. Environment Variables

| Variable | Use Case | Example |
|---|---|---|
| **`TF_LOG`** | Controls verbose logging level. | `export TF_LOG=DEBUG` (values: TRACE, DEBUG, INFO, WARN, ERROR) |
| **`TF_LOG_PATH`** | Output log locations. | `export TF_LOG_PATH=./terraform.log` |
| **`TF_VAR_<name>`** | Inject values into input variables. | `export TF_VAR_db_password="supersecurepassword"` |
| **`TF_DATA_DIR`** | Change location of `.terraform/` cache directory. | `export TF_DATA_DIR="/tmp/tf-cache"` |

# Terraform Terminology & Glossary

Glossary of key concepts, HCL blocks, operational terminology, and advanced refactoring primitives used in Terraform and Infrastructure-as-Code development.

---

## 1. Core HCL Syntax & Language Primitives

- **Resource**: The fundamental block that defines a physical or virtual infrastructure component (e.g., VPC, EC2 instance, DB instance).
- **Data Source (`data`)**: Queries external systems or existing live cloud infrastructure to fetch read-only attributes for use in code.
- **Provider**: A compiled Go plugin that translates HCL resource declarations into specific cloud API HTTP/REST requests.
- **Provider Alias**: Alternate provider configuration enabling multi-region or multi-account resource declarations within a single module.
- **Module**: A directory containing `.tf` files. The root directory is the *root module*; referenced subdirectories are *child modules*.
- **Variable (Input Variable)**: Configurable parameters passed into modules or configurations.
- **Output**: Return values exported by modules after deployment.
- **Local (Local Values)**: Scoped intermediate variables used inside a module to simplify HCL expressions and prevent duplication.
- **Lifecycle Block**: Meta-arguments nested in resources to control creation/destruction mechanics (`create_before_destroy`, `prevent_destroy`, `ignore_changes`).
- **`templatefile()`**: Native HCL function for rendering external template files (e.g., shell scripts) with dynamic variables.

---

## 2. Declarative Refactoring & Validation Primitives

- **`moved` Block (TF 1.1+)**: Declarative HCL block used to rename or relocate resources in state without destroying live infrastructure.
- **`import` Block (TF 1.5+)**: Declarative HCL block used to bring existing "brownfield" infrastructure under Terraform state control.
- **`check` Block (TF 1.5+)**: Declarative HCL block that executes continuous, non-blocking operational health assertions post-deployment.
- **`removed` Block (TF 1.7+)**: Declarative HCL block used to remove a resource from state tracking without triggering cloud API destruction.
- **`precondition`**: Lifecycle guardrail that validates inputs or dependencies *before* resource evaluation/creation.
- **`postcondition`**: Lifecycle guardrail that validates resource attributes *after* creation (e.g., asserting storage encryption).
- **Provider Lock File (`.terraform.lock.hcl`)**: Dependency lock file recording exact version numbers and cryptographic checksums of provider binaries.

---

## 3. State & Secrets Management

- **State File (`terraform.tfstate`)**: JSON database acting as Terraform's memory, mapping HCL identifiers to real-world cloud API resource IDs.
- **Remote Backend**: Hosted storage mechanism (e.g. S3, GCS, HCP Terraform) where state files reside to enable team collaboration.
- **State Locking**: Database mechanism (DynamoDB, native remote locks) preventing concurrent `terraform apply` operations from corrupting state.
- **State Drift**: Discrepancy between live cloud infrastructure properties and the recorded configuration in state (caused by out-of-band console edits).
- **Workspaces**: Isolated parallel state files within a single code directory. Discouraged for production environment isolation.
- **Secrets Management**: Strategies for injecting credentials (environment variables, Vault, AWS Secrets Manager) without committing plain-text secrets to Git.
- **`sensitive = true`**: Attribute flag that redacts variable values from terminal `stdout` output (does NOT encrypt state JSON).

---

## 4. Advanced Iteration & Expressions

- **`count`**: Integer-based loop construct best suited for simple boolean enable/disable toggles (`count = var.enable ? 1 : 0`).
- **`for_each`**: Map/Set-based loop construct creating resources with stable string keys (`aws_iam_user.example["alice"]`).
- **`for` Expression**: Expression syntax used to iterate over and transform collections inside locals or module arguments.
- **`dynamic` Block**: Construct used to iterate over collections and generate repeating *nested blocks* (e.g., `ingress` rules) inside resources.
- **`try()` / `can()`**: Built-in error-handling functions used to evaluate fallback expressions or check expression validity.

---

## 5. Operations & Team Workflows

- **Idempotency**: Guarantee that executing the same code multiple times produces the exact same end state without side effects.
- **Declarative**: Specifying *what* target infrastructure should look like, leaving the engine to calculate diffs and execution steps.
- **Blast Radius**: The potential scope of infrastructure impacted or destroyed by a single deployment execution.
- **Immutable Infrastructure**: Strategy where servers are replaced completely (destroyed and redeployed) rather than patched in place.
- **Atlantis**: Open-source pull-request automation server for Terraform GitOps workflows.
- **OIDC (OpenID Connect)**: Zero-trust authentication protocol allowing CI/CD runners to assume cloud IAM roles without static secret keys.
- **Policy-as-Code**: Enforcing governance and compliance rules (Sentinel / OPA) before infrastructure changes are applied.
- **Terratest**: Go library used to write automated integration tests that deploy, validate, and destroy real infrastructure.

# Terraform Terminology & Glossary

Glossary of key concepts, HCL blocks, and operational terminology used in Terraform and Infrastructure-as-Code development.

## 1. Core HCL Syntax & Blocks

- **Resource**: The fundamental block that defines a physical or virtual infrastructure component (e.g. Virtual Network, VM instance).
- **Data Source**: Queries external systems or APIs to fetch read-only data for use within HCL (e.g., retrieving an AMI ID or VPC metadata).
- **Provider**: A plugin that translates HCL definitions into specific cloud provider API calls (e.g. AWS, GCP, Kubernetes, GitHub).
- **Module**: A folder containing `.tf` files. Used to package and reuse groups of resources.
- **Variable (Input Variable)**: Parameters passed into modules or configurations to customize deployment settings.
- **Output**: Return values exported by modules or configuration roots after deployment.
- **Local (Local Values)**: Scoped intermediate variables used inside a module to simplify HCL expressions and prevent duplication.
- **Lifecycle Block**: Meta-arguments nested in resources to control creation/destruction mechanics (e.g., `create_before_destroy`, `prevent_destroy`, `ignore_changes`).

## 2. State & State Management

- **State File (`terraform.tfstate`)**: A JSON document that acts as Terraform's memory, mapping your code definitions to real deployed resources.
- **Remote Backend**: A remote storage mechanism (e.g. Amazon S3, Google Cloud Storage, HashiCorp Cloud) where the state file is hosted to enable team access.
- **State Locking**: A mechanism (using locking databases like DynamoDB or native remote locks) that blocks concurrent writes to a state file, preventing conflicts.
- **State Drift**: The divergence between the real-world infrastructure properties and the configuration recorded in the state file (usually caused by manual edits in the cloud console).
- **Workspaces**: Isolated parallel states managed within a single workspace folder. Useful for dev tests, but discouraged for production environment separation.

## 3. IaC Operations

- **Idempotency**: The guarantee that running the same deployment command multiple times produces the exact same end state without side effects.
- **Declarative Programming**: Specifying *what* the final target infrastructure looks like (e.g., "I want 3 servers"), leaving the engine to resolve the steps.
- **Procedural Programming**: Specifying the step-by-step commands to run (e.g., "Run apt-get update, start VM, configure proxy").
- **Blast Radius**: The amount of infrastructure that would be impacted or destroyed if a single module failed or was deleted.
- **Immutable Infrastructure**: A practice where servers are replaced completely (destroyed and redeployed) rather than modified in place when configuration changes.
- **Dry Run**: Previewing what changes will be executed on the real infrastructure before running the command (achieved via `terraform plan`).

--- 
name: terraform-up-running
description: >
  Infrastructure-as-Code mastery from "Terraform: Up & Running" (3rd ed., O'Reilly 2023)
  by Yevgeniy Brikman. Covers HCL syntax, state management (S3/DynamoDB), composable modules,
  secrets management (Vault/AWS SM), loops/conditionals, refactoring (moved/import/check),
  production-grade architecture, native HCL testing & Terratest, OIDC CI/CD pipelines, and
  team workflows. Opinionated, production-ready guidance distilled into an agent instruction set.

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
  - moved block
  - import block
  - check block
  - secrets management
  - terratest
---

# Terraform: Up & Running — Agent Skill & Reference

> *"Terraform is a tool for building, changing, and versioning infrastructure safely and efficiently."*  
> — Yevgeniy Brikman, *Terraform: Up & Running*, 3rd Edition (O'Reilly, 2023)

## How to Use This Skill

When tasked with generating, reviewing, or refactoring Terraform code or designing team infrastructure workflows, follow the decision frameworks and strict code generation rules below.

| Need | Go To |
|---|---|
| Enforceable HCL rules | `agent-rules.md` |
| Decision trees & CLI commands | `cheatsheet.md` |
| Terminology & definitions | `glossary.md` |
| Architecture & design patterns | `patterns.md` |
| Deep chapter guidance | `chapters/ch0N.md` |

---

## AGENT CODE GENERATION RULES (MANDATORY)

When generating or editing Terraform HCL code, you **MUST** strictly adhere to the following rules:

1. **State Isolation**: Never use `terraform workspace` for production environment separation. Use dedicated directory trees (`stage/`, `prod/`).
2. **Provider Versioning**: Always declare `required_providers` with pessimistic version constraints (`~> 5.0`) and commit `.terraform.lock.hcl`.
3. **Secrets Security**: NEVER place plain-text secrets in code or `.tfvars`. Query AWS Secrets Manager/Vault dynamically or inject via `TF_VAR_<name>`.
4. **Stable Iteration**: Always prefer `for_each` over `count` for resource iteration. Use `count` ONLY for simple boolean toggles (`count = var.enable ? 1 : 0`).
5. **Declarative Refactoring**: Use `moved {}` blocks for resource renames/module moves and `import {}` blocks for brownfield onboarding. Never suggest manual `terraform state mv` scripts.
6. **Extensible Security Groups**: Standardize on standalone rule resources (`aws_security_group_rule`) instead of inline `ingress {}` blocks in reusable modules.
7. **Module Pathing**: Always load scripts/templates relative to `${path.module}` inside child modules (e.g. `${path.module}/user_data.sh.tpl`).
8. **Contract Enforcement**: Include `description` on ALL variables and outputs, and add `validation` blocks to inputs and `precondition`/`postcondition` to critical resources.
9. **Zero-Downtime Rollover**: Pair Launch Templates and ASGs with `create_before_destroy = true` and dynamic name prefixes.
10. **Plan Artifact CI/CD**: Always execute `terraform plan -out=tfplan` and apply ONLY the saved `tfplan` artifact via OIDC authentication.

---

## Decision Framework: Routing Tasks to Chapters

```
What is the user trying to accomplish?
├── Why IaC / Tool Selection / Architecture comparison?   → Read chapters/ch01.md
├── Basic HCL / EC2 / ALB / ASG / Variables / Outputs?   → Read chapters/ch02.md
├── Remote State / S3 + DynamoDB / State Isolation?     → Read chapters/ch03.md
├── Building Reusable Modules / Inputs / Outputs / Locals? → Read chapters/ch04.md
├── Loops / for_each / dynamic blocks / ASG Rollover?     → Read chapters/ch05.md
├── Secrets Management / Vault / AWS Secrets Manager?     → Read chapters/ch06.md
├── Multi-Region / Multi-Account / AssumeRole / K8s?       → Read chapters/ch07.md
├── Refactoring / moved / import / check / Lock files?    → Read chapters/ch08.md
├── Unit Testing / terraform test / Terratest / Mocks?     → Read chapters/ch09.md
└── Team CI/CD / Atlantis / OIDC / Policy-as-Code (OPA)?  → Read chapters/ch10.md
```

---

## Chapter Index

| Ch | Title | Key Frameworks & Topics Covered |
| --- | ---| ---|
| [ch01](chapters/ch01.md) | Why Terraform | 5 IaC categories, Declarative vs Procedural, Immutable vs Mutable, Agentless vs Agent |
| [ch02](chapters/ch02.md) | Getting Started | HCL anatomy, Providers, Launch Templates, ALB + ASG cluster, `templatefile()` |
| [ch03](chapters/ch03.md) | Managing State | S3 + DynamoDB locking, File directory layout, `terraform_remote_state` |
| [ch04](chapters/ch04.md) | Reusable Modules | Module inputs/outputs/locals, `${path.module}`, Standalone resources vs Inline blocks |
| [ch05](chapters/ch05.md) | Loops & Conditionals | `count` vs `for_each` vs `for` vs `dynamic`, string directives, Zero-downtime ASG |
| [ch06](chapters/ch06.md) | Managing Secrets | Secrets Manager, Vault, `TF_VAR_`, State leak warnings, `precondition`/`postcondition` |
| [ch07](chapters/ch07.md) | Multiple Providers | Provider aliases, Multi-account AssumeRole, `configuration_aliases`, Kubernetes |
| [ch08](chapters/ch08.md) | Production Code | `moved` blocks, `import` blocks, `check` blocks, `removed` blocks, `.terraform.lock.hcl` |
| [ch09](chapters/ch09.md) | Testing Terraform | Testing pyramid, `tflint`/`checkov`, Native `terraform test` & Mocks, `Terratest` Go |
| [ch10](chapters/ch10.md) | Team Workflows | PR lifecycle, Atlantis GitOps, GitHub Actions OIDC pipeline, OPA/Sentinel policies |

---

## Topic Index

- **Aliased Providers**: [ch07](chapters/ch07.md)
- **Atlantis**: [ch10](chapters/ch10.md)
- **AssumeRole**: [ch07](chapters/ch07.md)
- **Auto Scaling Group (ASG)**: [ch02](chapters/ch02.md), [ch05](chapters/ch05.md)
- **Backend (S3 + DynamoDB)**: [ch03](chapters/ch03.md)
- **Blast Radius**: [ch01](chapters/ch01.md), [ch03](chapters/ch03.md)
- **Check Blocks**: [ch08](chapters/ch08.md)
- **Checkov / TFSec**: [ch09](chapters/ch09.md)
- **Count vs For_Each**: [ch05](chapters/ch05.md)
- **Declarative Refactoring**: [ch08](chapters/ch08.md)
- **Directory Layout**: [ch03](chapters/ch03.md)
- **Dynamic Blocks**: [ch05](chapters/ch05.md)
- **Import Blocks**: [ch08](chapters/ch08.md)
- **Launch Templates**: [ch02](chapters/ch02.md)
- **Lock File (.terraform.lock.hcl)**: [ch08](chapters/ch08.md)
- **Modules**: [ch04](chapters/ch04.md), [ch08](chapters/ch08.md)
- **Moved Blocks**: [ch08](chapters/ch08.md)
- **OIDC Authentication**: [ch10](chapters/ch10.md)
- **Open Policy Agent (OPA)**: [ch10](chapters/ch10.md)
- **Path.module**: [ch04](chapters/ch04.md)
- **Precondition & Postcondition**: [ch06](chapters/ch06.md)
- **Removed Blocks**: [ch08](chapters/ch08.md)
- **Secrets Management**: [ch06](chapters/ch06.md)
- **Sensitive Variables**: [ch06](chapters/ch06.md)
- **State Locking**: [ch03](chapters/ch03.md)
- **Templatefile**: [ch02](chapters/ch02.md), [ch04](chapters/ch04.md)
- **Terraform Test (HCL Unit Tests)**: [ch09](chapters/ch09.md)
- **Terratest (Go)**: [ch09](chapters/ch09.md)
- **TFLint**: [ch09](chapters/ch09.md)
- **Vault (HashiCorp)**: [ch06](chapters/ch06.md)
- **Workspaces Gotchas**: [ch03](chapters/ch03.md)
- **Zero-Downtime Deployment**: [ch02](chapters/ch02.md), [ch05](chapters/ch05.md)

---

## Supporting Files

- [agent-rules.md](agent-rules.md) — Hard rules for generating production-grade Terraform HCL
- [cheatsheet.md](cheatsheet.md) — Decision trees, command reference, and CLI flags
- [glossary.md](glossary.md) — 30+ core & advanced Terraform terminology definitions
- [patterns.md](patterns.md) — 9 production engineering architecture & security patterns

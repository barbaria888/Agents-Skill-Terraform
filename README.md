<img width="1500" height="27" alt="seperator" src="https://github.com/user-attachments/assets/4d6c5f41-bf43-4f0c-8768-60582937ad58" />

# <img  height="70" alt="opentofu" src="https://github.com/user-attachments/assets/f8534073-a35b-4273-9cc5-274dd60dea21" /><img  height="70" alt="terraform" src="https://github.com/user-attachments/assets/9b7da754-a646-4f5b-b775-bb750747bd1f" /> Terraform: Up & Running - AI Knowledge Skill

> *"Terraform is a tool for building, changing, and versioning infrastructure safely and efficiently."*
> — Yevgeniy Brikman, *Terraform: Up & Running* (3rd Edition, O'Reilly, 2023)

<p align="center">
  <img 
    src="https://capsule-render.vercel.app/api?type=rect&color=0:5C4EE5,50:7B42BC,100:844FBA&height=3&section=header"
    width="100%"
  />
</p>

## What Is This?

This repository is a **structured AI knowledge skill** synthesized from the O'Reilly book
[*Terraform: Up & Running: Writing Infrastructure as Code* (3rd Edition)](https://www.oreilly.com/library/view/terraform-up-and/9781098116736/)
by Yevgeniy Brikman.

It was built using the [**book-to-skill**](https://github.com/virgiliojr94/book-to-skill)
methodology — transforming technical books into **agent-ready,
token-efficient knowledge repositories** that AI coding assistants can load
on demand to give you expert-level, production-grade guidance.

This is **not** a book summary. It is a living engineering reference:
- **Core Mental Models**: The Terraform lifecycle (`init` → `plan` → `apply` → `destroy`) and state isolation strategies
- **HCL Syntax & Patterns**: Resource declarations, variables, outputs, locals, and data sources
- **Advanced Expressions**: Loops (`count`, `for_each`, `for`), conditional expressions, and `dynamic` blocks
- **Production Architecture**: S3/DynamoDB remote state locking, zero-downtime ASG rollouts, and composable modules
- **Multi-Provider & Multi-Region**: Provider aliases, multi-account assume-role setups, and Kubernetes orchestration
- **Automated Testing**: Static analysis (`tflint`, `tfsec`), native HCL unit testing (`terraform test`), and Go integration tests (`Terratest`)
- **Team Workflows**: Pull request review guidelines, Atlantis GitOps automation, and Terraform Cloud

<p align="center">
  <img 
    src="https://capsule-render.vercel.app/api?type=rect&color=0:5C4EE5,50:7B42BC,100:844FBA&height=3&section=header"
    width="100%"
  />
</p>

## 📚 Repository Map

```
.
├── SKILL.md             # Primary Agent Skill registration and index
├── cheatsheet.md        # CLI commands, state commands, flags & debug vars
├── glossary.md          # 20+ core terms (HCL blocks, state terms, operations)
├── patterns.md          # Production architecture & directory layout patterns
├── chapters/
│   ├── ch01.md          # Why Terraform & IaC Categories
│   ├── ch02.md          # Getting Started (Single Server, Cluster, ALB)
│   ├── ch03.md          # Managing State (S3 + DynamoDB, Isolation)
│   ├── ch04.md          # Reusable Modules (Inputs, Outputs, Locals, Gotchas)
│   ├── ch05.md          # Loops, Conditionals, Zero-Downtime Deployments
│   ├── ch06.md          # Production-Grade Infrastructure
│   ├── ch07.md          # Multiple Providers, Regions & Accounts
│   ├── ch08.md          # Production-Grade Modules (Versioning & Registry)
│   ├── ch09.md          # Testing Terraform Code (Terratest & Unit Tests)
│   └── ch10.md          # Team Workflows, CI/CD, Atlantis & TF Cloud
└── README.md            # Documentation & quick start guide
```

<p align="center">
  <img 
    src="https://capsule-render.vercel.app/api?type=rect&color=0:5C4EE5,50:7B42BC,100:844FBA&height=3&section=header"
    width="100%"
  />
</p>

# How this helps you?

<p align="center">
  <img 
    src="https://capsule-render.vercel.app/api?type=rect&color=0:5C4EE5,50:7B42BC,100:844FBA&height=3&section=header"
    width="100%"
  />
</p>

### Using This Skill with AI Coding Assistants

This repository follows the **book-to-skill** methodology and is designed to be consumed by AI coding assistants that support reusable Agent Skills or can use a local repository as context.

---

## 1. Clone the Repository

Clone this repository to your local machine:

```bash
git clone https://github.com/barbaria888/Agents-Skill-Terraform.git
cd Agents-Skill-Terraform
```

<p align="center">
  <img 
    src="https://capsule-render.vercel.app/api?type=rect&color=0:5C4EE5,50:7B42BC,100:844FBA&height=3&section=header"
    width="100%"
  />
</p>

## Claude Code

Install the repository as a local Agent Skill:

```bash
git clone https://github.com/barbaria888/Agents-Skill-Terraform.git \
~/.claude/skills/terraform-up-running
```

Start a new Claude Code session.

Example prompts:

```text
Design a zero-downtime ASG rollout pattern using the guidance in this skill.

Explain the trade-offs between workspaces and file layout for state isolation.

Show me how to write a Terratest integration test for an ALB module.

Write a reusable module with required provider configuration aliases.
```

<p align="center">
  <img 
    src="https://capsule-render.vercel.app/api?type=rect&color=0:5C4EE5,50:7B42BC,100:844FBA&height=3&section=header"
    width="100%"
  />
</p>

## GitHub Copilot CLI

Install the repository into your skills directory:

```bash
git clone https://github.com/barbaria888/Agents-Skill-Terraform.git \
~/.copilot/skills/terraform-up-running
```

Reload your skills:

```text
/skills reload
```

The skill will now appear in your available skills.

<p align="center">
  <img 
    src="https://capsule-render.vercel.app/api?type=rect&color=0:5C4EE5,50:7B42BC,100:844FBA&height=3&section=header"
    width="100%"
  />
</p>

## Antigravity / Gemini CLI

Antigravity natively discovers skills located in the workspace `.agents/` folder.

When working in this repository or including it in your project workspace:

Recommended prompt:

```text
Use this repository as the authoritative reference.

Read SKILL.md first.

Use the chapter files, glossary, patterns, and cheatsheet before relying on general knowledge.
```

<p align="center">
  <img 
    src="https://capsule-render.vercel.app/api?type=rect&color=0:5C4EE5,50:7B42BC,100:844FBA&height=3&section=header"
    width="100%"
  />
</p>

## Other AI Coding Assistants (Cursor, Windsurf, Roo Code, Aider)

Any assistant capable of reading a local repository or Markdown documentation can use this project as a structured knowledge base.

Recommended workflow:

1. Read `SKILL.md`.
2. Navigate to the relevant chapter in `chapters/`.
3. Use `patterns.md` for production architecture patterns.
4. Use `glossary.md` for terminology definitions.
5. Use `cheatsheet.md` for commands, debugging flags, and CLI parameters.

<p align="center">
  <img 
    src="https://capsule-render.vercel.app/api?type=rect&color=0:5C4EE5,50:7B42BC,100:844FBA&height=3&section=header"
    width="100%"
  />
</p>

> **Note**
>
> This repository contains structured engineering knowledge—not executable infrastructure code or deployment runbooks.
>
> Native Agent Skill support depends on your AI coding assistant.
> Other assistants can still use the repository effectively by loading it as project context.

<img width="1500" height="27" alt="seperator" src="https://github.com/user-attachments/assets/4d6c5f41-bf43-4f0c-8768-60582937ad58" />

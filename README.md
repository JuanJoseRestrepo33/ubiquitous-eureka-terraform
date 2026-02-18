# Terraform Practice Modules

This repository organizes Terraform infrastructure using a structure based on **reusable modules**, **deployable stacks**, and **environment-specific configuration**.  
The main goal is to keep responsibilities separated, simplify maintenance, and avoid state collisions.

## Why this folder structure?

- **`terraform/modules`**: contains reusable components (for example, a module to create an ECR repository).
- **`terraform/stacks`**: contains deployable assemblies that use modules to provision cloud infrastructure.
- **`terraform/environments`**: contains per-environment parameters (remote backend and input variables).

> The **Stack** concept is not native to Terraform; it is an architecture convention used to organize code more clearly.

## Use Case: Container Registry (ECR)

In this project, the `repository` module was created to provision an **Amazon ECR** repository.

### Important Best-Practice Note

In this academic exercise, the configuration uses:
- **mutable** repositories (tags can be overwritten), and
- image scanning disabled.

⚠️ **BAD PRACTICE FOR PRODUCTION**: in real projects, always enable image immutability and image scanning to reduce security risk.

## Typical Files in a Terraform Module/Stack

| File | Description |
| --- | --- |
| `locals.tf` | Constants and helper expressions used to build internal module values. |
| `data.tf` | `data` blocks used to query existing provider information. |
| `main.tf` | Main file containing `resource` blocks (infrastructure). |
| `outputs.tf` | `output` blocks defining values exposed by the module/stack. |
| `variables.tf` | `variable` blocks defining input arguments (inputs). |

## Environment Files

| File | Description |
| --- | --- |
| `backend.tfvars` | Remote backend parameters where Terraform state (`tfstate`) is stored. |
| `terraform.tfvars` | Variable values used to parameterize the stack for a specific environment. |

## Remote State (Backend)

For `container_registry`, state is stored in S3 using a key such as:

- `container_registry/terraform.tfstate`

This key (`key`) must be **unique per stack**. Reusing it in another stack causes state collisions.

Logical S3 location example:

- `s3://<bucket>/container_registry/terraform.tfstate`

## Recommended Flow: init, validate, plan, apply

> On Windows/Git Bash (or whenever the path contains spaces), use quotes around `$(pwd)`.

### 1) Initialize backend and providers

```bash
terraform -chdir="$(pwd)/terraform/stacks/container_registry" \
  init \
  -backend-config="$(pwd)/terraform/environments/<username>/container_registry/backend.tfvars"
```

### 2) Validate configuration

```bash
terraform -chdir="$(pwd)/terraform/stacks/container_registry" validate
```

### 3) Generate execution plan

```bash
terraform -chdir="$(pwd)/terraform/stacks/container_registry" \
  plan \
  -var-file="$(pwd)/terraform/environments/<username>/container_registry/terraform.tfvars" \
  -out="$(pwd)/terraform/stacks/container_registry/.tfplan"
```

### 4) Apply plan

```bash
terraform -chdir="$(pwd)/terraform/stacks/container_registry" \
  apply "$(pwd)/terraform/stacks/container_registry/.tfplan"
```

## Suggested Convention for Multiple Stacks/Environments

A clear state-key convention makes scaling easier:

- `<environment>/<stack>/terraform.tfstate`

Examples:
- `dev/container_registry/terraform.tfstate`
- `prod/container_registry/terraform.tfstate`

## Official Terraform References

- Locals: https://developer.hashicorp.com/terraform/language/values/locals
- Data Sources: https://developer.hashicorp.com/terraform/language/data-sources
- Resources: https://developer.hashicorp.com/terraform/language/resources
- Outputs: https://developer.hashicorp.com/terraform/language/values/outputs
- Variables: https://developer.hashicorp.com/terraform/language/values/variables
- Plan: https://developer.hashicorp.com/terraform/cli/commands/plan

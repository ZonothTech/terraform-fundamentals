# Lab 02 - Organize your code, resources & state

> Consolidates **Session 02 — Main Concepts**: the CLI, providers & versions, resources
> and dependencies, and Terraform state. You keep working in your `terraform-training`
> project from Lab 01.

## Table of Contents

- [Goals](#goals)
- [Pre-requisites](#pre-requisites)
- [Guide](#guide)
  - [Step 01: Split your code into files](#step-01-split-your-code-into-files)
  - [Step 02: Pin the provider version](#step-02-pin-the-provider-version)
  - [Step 03: Add a second resource with a dependency](#step-03-add-a-second-resource-with-a-dependency)
  - [Step 04: Format, validate and init](#step-04-format-validate-and-init)
  - [Step 05: Plan with a saved plan file](#step-05-plan-with-a-saved-plan-file)
  - [Step 06: Apply and inspect the state](#step-06-apply-and-inspect-the-state)
  - [Step 07: The lock file](#step-07-the-lock-file)
- [Conclusion](#conclusion)

## Goals

- Organize Terraform code the way the community does (`main` / `versions` / `outputs`)
- Understand provider version constraints
- See how referencing one resource from another creates an implicit dependency
- Read Terraform state and understand the lock file

## Pre-requisites

- Have finished [Lab 01](lab01.md) — keep working in the same `terraform-training` folder
- GitHub CLI authenticated (`gh auth login`)

## Guide

### Step 01: Split your code into files

Terraform merges every `.tf` file in the folder, but the community follows one layout.
Create these files and move the matching blocks out of `main.tf`.

`versions.tf`:

```hcl
terraform {
  required_version = ">= 1.9"
  required_providers {
    github = {
      source  = "integrations/github"
      version = "~> 6.0"
    }
  }
}

provider "github" {}
```

`main.tf` keeps only the resources:

```hcl
resource "github_repository" "course_repo" {
  name        = "terraform-training"
  description = "My Terraform training project"
  visibility  = "public"
  has_issues  = true
  has_wiki    = false
}
```

Create an empty `outputs.tf` for now.

### Step 02: Pin the provider version

Look at the version constraint `~> 6.0`. Recall the operators from the session:

- `~> 6.0` allows any `6.x`, but not `7.0`
- `>= 6.2` allows anything from 6.2 upward
- `= 6.3.1` pins one exact version

Leave it at `~> 6.0` — a safe default that takes patch and minor updates but no surprise majors.

### Step 03: Add a second resource with a dependency

Add an issue label to the repository. Notice it **references** the repository resource —
that reference is what makes Terraform create the repo first.

```hcl
resource "github_issue_label" "bug" {
  repository = github_repository.course_repo.name   # implicit dependency
  name       = "erro"
  color      = "d73a4a"
}
```

The reference pattern is `type.name.attribute` — here `github_repository.course_repo.name`.

### Step 04: Format, validate and init

```bash
terraform fmt
terraform init
terraform validate
```

`fmt` rewrites the files to canonical style; `init` downloads the provider into
`.terraform`; `validate` checks the syntax without contacting GitHub.

### Step 05: Plan with a saved plan file

```bash
terraform plan -out mainPlan
terraform show mainPlan
```

Read the plan: Terraform will add the repository and the label. Because the label
references the repository, they are created in the right order automatically.

### Step 06: Apply and inspect the state

```bash
terraform apply mainPlan
```

Now explore the state — Terraform's memory of what it built:

```bash
terraform state list                          # every managed resource
terraform state show github_repository.course_repo   # its stored attributes
```

Never edit `terraform.tfstate` by hand — only Terraform should write it.

### Step 07: The lock file

Open `.terraform.lock.hcl`. It pins the exact provider version and hashes that `init`
selected. Commit it like source code so everyone uses the same provider version.

## Conclusion

You organized your project into the standard files, saw how references drive dependency
ordering, and read both the state and the lock file — the core mechanics behind every
Terraform run. Next session you'll make this code dynamic with variables.

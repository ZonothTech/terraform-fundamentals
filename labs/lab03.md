# Lab 03 - Variables, locals & outputs

> Consolidates **Session 03 — Variables & Outputs**: input variables and types, setting
> values and precedence, validation, locals, and outputs (including sensitive ones).

## Table of Contents

- [Goals](#goals)
- [Pre-requisites](#pre-requisites)
- [Guide](#guide)
  - [Step 01: Declare input variables](#step-01-declare-input-variables)
  - [Step 02: Use the variables in your code](#step-02-use-the-variables-in-your-code)
  - [Step 03: Add locals](#step-03-add-locals)
  - [Step 04: Add outputs](#step-04-add-outputs)
  - [Step 05: Override with the command line](#step-05-override-with-the-command-line)
  - [Step 06: Use a tfvars file](#step-06-use-a-tfvars-file)
  - [Step 07: Trigger the validation rule](#step-07-trigger-the-validation-rule)
  - [Step 08: Apply and read outputs](#step-08-apply-and-read-outputs)
- [Conclusion](#conclusion)

## Goals

- Declare typed input variables with defaults, descriptions and validation
- Use variables and locals in your code
- Understand the variable precedence order
- Return values with outputs, including sensitive ones

## Pre-requisites

- Have finished [Lab 02](lab02.md) — same `terraform-training` project
- GitHub CLI authenticated

## Guide

### Step 01: Declare input variables

In `variables.tf`, add:

```hcl
variable "repo_name" {
  description = "The name of the repository"
  type        = string
  default     = "terraform-training"
  validation {
    condition     = length(var.repo_name) <= 20
    error_message = "The repository name must be 20 characters or fewer."
  }
}

variable "repo_private" {
  description = "Whether the repository is private"
  type        = bool
  default     = true
}

variable "welcome_message" {
  description = "Optional welcome message posted as an issue"
  type        = string
  default     = null
}
```

### Step 02: Use the variables in your code

Update the repository resource in `main.tf` to read from the variables, and add a
conditional issue driven by `welcome_message`:

```hcl
resource "github_repository" "course_repo" {
  name       = var.repo_name
  visibility = var.repo_private ? "private" : "public"
  has_issues = true
  has_wiki   = false
}

resource "github_issue" "welcome" {
  count      = var.welcome_message != null ? 1 : 0
  repository = github_repository.course_repo.name
  title      = "Welcome to ${github_repository.course_repo.name}"
  body       = var.welcome_message
}
```

### Step 03: Add locals

Locals are values you compute once and reuse. Add a `locals` block:

```hcl
locals {
  common_labels = ["bug", "enhancement", "question"]
}

resource "github_issue_label" "labels" {
  for_each   = toset(local.common_labels)
  repository = github_repository.course_repo.name
  name       = each.value
  color      = "ededed"
}
```

### Step 04: Add outputs

In `outputs.tf`:

```hcl
output "repo_url" {
  value = github_repository.course_repo.html_url
}

output "repo_clone_url" {
  description = "HTTPS clone URL"
  value       = github_repository.course_repo.http_clone_url
  sensitive   = true
}
```

`repo_clone_url` is marked `sensitive`, so Terraform will redact it in normal output.

### Step 05: Override with the command line

The command line has the highest precedence. Preview creating the welcome issue:

```bash
terraform plan -var="welcome_message=Welcome to my repo!" -out lastPlan
```

### Step 06: Use a tfvars file

Create `course.tfvars` for environment-style values:

```hcl
repo_private    = true
welcome_message = "Welcome to my repo!"
```

```bash
terraform plan -var-file="course.tfvars" -out lastPlan
```

Remember the precedence: **CLI `-var` > tfvars > `TF_VAR_` env var > default > prompt.**

### Step 07: Trigger the validation rule

Prove the validation works by passing a name that's too long:

```bash
terraform plan -var="repo_name=terraform-training-project-123" -out lastPlan
```

You'll get an error from your validation rule. Drop the value back to something valid.

### Step 08: Apply and read outputs

```bash
terraform apply lastPlan
terraform output                       # non-sensitive values print
terraform output repo_clone_url        # reveal the sensitive one on demand
```

Commit your work:

```bash
git add . && git commit -m "Lab 03: variables, locals and outputs" && git push
```

## Conclusion

Your project is now dynamic: typed and validated inputs, computed locals, a conditional
resource, and outputs that expose exactly what you want (and hide what you don't).

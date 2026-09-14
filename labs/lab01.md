# Lab 01 - Run your first Terraform commands

> **This is your course project.** The repository and folder you create here are reused
> and extended in every later lab. Keep them!

## Contents

- [Objectives](#objectives)
- [Pre-requisites](#pre-requisites)
- [Guide](#guide)
  - [Step 01: Create your project folder](#step-01-create-your-project-folder)
  - [Step 02: Author your first Terraform code](#step-02-author-your-first-terraform-code)
  - [Step 03: Initialize Terraform](#step-03-initialize-terraform)
  - [Step 04: Validate the code](#step-04-validate-the-code)
  - [Step 05: Plan the changes](#step-05-plan-the-changes)
  - [Step 06: Login to GitHub](#step-06-login-to-github)
  - [Step 07: Plan again with a saved plan](#step-07-plan-again-with-a-saved-plan)
  - [Step 08: Apply the changes](#step-08-apply-the-changes)
  - [Step 09: Inspect the state](#step-09-inspect-the-state)
  - [Step 10: Destroy the resource](#step-10-destroy-the-resource)
  - [Step 11: Re-authenticate on GitHub](#step-11-re-authenticate-on-github)
  - [Step 12: Run destroy again](#step-12-run-destroy-again)

## Objectives

- Author your first Terraform code
- Initialize, validate, plan and apply
- Read Terraform state
- Understand that Terraform always depends on provider auth

## Pre-requisites

- You completed [Lab 00 – Course setup](lab00-setup.md): Terraform, GitHub CLI and a
  code editor are installed, and you have a GitHub account.

> If `terraform --version` or `gh --version` fails, go back to
> [Lab 00](lab00-setup.md) before continuing.

## Guide

### Step 01: Create your project folder

Create the folder that will hold your course project for the whole training:

```bash
mkdir terraform-training
cd terraform-training
```

### Step 02: Author your first Terraform code

Create a file named `main.tf` with the following content:

```hcl
terraform {
  required_providers {
    github = {
      source  = "integrations/github"
      version = "~> 6.0"
    }
  }
}

# Configure the GitHub Provider
provider "github" {}

# Create a GitHub Repository
resource "github_repository" "course_repo" {
  name        = "terraform-training"
  description = "My Terraform training project"
  visibility  = "private"

  # Optional settings
  has_issues = true
  has_wiki   = false
}
```

Format the code and confirm it rewrites the file cleanly:

```bash
terraform fmt
```

This code will create a repository named `terraform-training` on your account —
the same repo you'll push your lab code to in later sessions.

### Step 03: Initialize Terraform

```bash
terraform init
```

Terraform downloads the GitHub provider into the `.terraform` folder and writes a
`.terraform.lock.hcl` file recording the exact version. **Commit the lock file** —
it guarantees everyone uses the same provider version.

This code uses the [GitHub Terraform Provider](https://registry.terraform.io/providers/integrations/github/latest/docs).

### Step 04: Validate the code

```bash
terraform validate
# Success! The configuration is valid.
```

Now break it on purpose to see how errors look:

- Delete the line `name = "terraform-training"`
- Add a line `has_popcorn = true` below `has_wiki = false`

Run validate again:

```bash
terraform validate
```

You'll get two errors: `name` is required, and `has_popcorn` is not a valid argument.
If you use VS Code with the Terraform extension, you'll also see them inline. Undo
both changes before continuing.

### Step 05: Plan the changes

```bash
terraform plan
```

You'll get a `401 Bad credentials` error — the provider needs to authenticate with
GitHub. We fix that next. (See other options in the
[provider auth docs](https://registry.terraform.io/providers/integrations/github/latest/docs#authentication).)

### Step 06: Login to GitHub

```bash
gh auth login
```

Choose GitHub.com → HTTPS → authenticate Git → login with a web browser, and enter
the one-time code. Verify:

```bash
gh auth status
```

### Step 07: Plan again with a saved plan

```bash
terraform plan
```

Read the output: `Plan: 1 to add, 0 to change, 0 to destroy.` Attributes marked
`(known after apply)` are values GitHub assigns at creation time.

Terraform recommends saving the plan. Do it, then inspect it:

```bash
terraform plan -out myplan
terraform show myplan     # human-readable view of the binary plan file
```

### Step 08: Apply the changes

```bash
terraform apply myplan
```

Because you passed a saved plan, Terraform applies exactly those actions with no
extra prompt. Check GitHub — your repository now exists. 🎉

### Step 09: Inspect the state

```bash
terraform state list
cat terraform.tfstate
```

The state file (JSON) is how Terraform remembers what it created. Never edit it by
hand.

### Step 10: Destroy the resource

```bash
terraform destroy
```

Confirm with `yes`. You'll get a `403 Must have admin rights` error — the default
token can't delete repos. That's the next step.

### Step 11: Re-authenticate on GitHub

```bash
gh auth login --scopes delete_repo
gh auth status
```

This grants the extra scope needed to delete the repository — a reminder that
Terraform always depends on the provider's auth and permissions.

### Step 12: Run destroy again

```bash
terraform destroy
```

Confirm with `yes`. The repository is deleted and the state is now empty.

> **Before Lab 02:** re-create the repo with `terraform apply` (you'll keep building
> on it), or simply move on — Lab 02 starts from this same folder.

## Conclusion

You authored, initialized, validated, planned, applied and destroyed real
infrastructure — and saw first-hand how Terraform relies on provider authentication.

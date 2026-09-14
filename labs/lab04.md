# Lab 04 - Conditionals & iterations

> Consolidates **Session 04 — Conditionals & Iterations**: `count`, `for_each`, `for`
> expressions and outputs over collections. Still on the GitHub provider — zero cloud cost.

## Table of Contents

- [Goals](#goals)
- [Pre-requisites](#pre-requisites)
- [Guide](#guide)
  - [Step 01: Drive labels from a map with for_each](#step-01-drive-labels-from-a-map-with-for_each)
  - [Step 02: Add a conditional resource with count](#step-02-add-a-conditional-resource-with-count)
  - [Step 03: Transform a collection with a for expression](#step-03-transform-a-collection-with-a-for-expression)
  - [Step 04: Output over the collection](#step-04-output-over-the-collection)
  - [Step 05: Apply and experiment](#step-05-apply-and-experiment)
- [Conclusion](#conclusion)

## Goals

- Use `for_each` to create many resources from a map
- Use `count` to conditionally create a resource
- Build a value with a `for` expression
- Reference and output resources created in a loop
- See why `for_each` keys are more stable than `count` indexes

## Pre-requisites

- Have finished [Lab 03](lab03.md) — same `terraform-training` project
- GitHub CLI authenticated

## Guide

### Step 01: Drive labels from a map with for_each

Replace the simple labels from Lab 03 with a **map** of name → color, and iterate it
with `for_each`. Use `each.key` for the name and `each.value` for the color.

In `variables.tf`:

```hcl
variable "labels" {
  type        = map(string)   # name => hex color
  description = "Issue labels to manage on the repo."
  default = {
    "bug"         = "d73a4a"
    "enhancement" = "a2eeef"
    "question"    = "d876e3"
  }
}

variable "enable_release_label" {
  type    = bool
  default = false
}
```

In `main.tf`:

```hcl
resource "github_issue_label" "labels" {
  for_each   = var.labels
  repository = github_repository.course_repo.name
  name       = each.key
  color      = each.value
}
```

### Step 02: Add a conditional resource with count

Create a `release` label only when a flag is on:

```hcl
resource "github_issue_label" "release" {
  count      = var.enable_release_label ? 1 : 0
  repository = github_repository.course_repo.name
  name       = "release"
  color      = "0e8a16"
}
```

### Step 03: Transform a collection with a for expression

A `for` expression builds a **value**, not resources. Add a local that upper-cases the
label names:

```hcl
locals {
  label_names_upper = [for name in keys(var.labels) : upper(name)]
}
```

### Step 04: Output over the collection

Because the labels were created with `for_each`, reference them by map key. In `outputs.tf`:

```hcl
output "managed_labels" {
  value = {
    for name, label in github_issue_label.labels :
    name => label.color
  }
}
```

### Step 05: Apply and experiment

```bash
terraform fmt
terraform validate
terraform plan -out lastPlan
terraform apply "lastPlan"
terraform output managed_labels
```

Now read each plan carefully as you change things:

- **Add** a key to the `labels` map and apply — only the new label is created.
- **Remove** a key and apply — only that label is destroyed; the others are untouched.
- Flip `enable_release_label=true` / `false` and apply.

> Try the same with `count` instead of `for_each`: removing a middle item forces
> Terraform to recreate the ones after it — the classic reason to prefer `for_each` for
> named resources.

Commit your work:

```bash
git add . && git commit -m "Lab 04: conditionals and iterations" && git push
```

## Conclusion

One map now drives any number of resources, a boolean toggles a resource on and off, and
an output summarizes the whole collection — all while keeping your code DRY. Next session
you leave GitHub behind and move to Azure.

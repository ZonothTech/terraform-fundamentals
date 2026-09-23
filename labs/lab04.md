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

Declare the variables in `variables.tf` — declarations only, no values:

```hcl
variable "labels" {
  type        = map(string) # name => hex color
  description = "Issue labels to manage on the repo."
}

variable "enable_release_label" {
  type    = bool
  default = false
}
```

Put the **values** in a `terraform.tfvars` file. Terraform loads it automatically, so you
change data there without ever touching `variables.tf`. This is the habit to build:
**code in `.tf`, values in `.tfvars`.**

```hcl
# terraform.tfvars
labels = {
  "bug"         = "d73a4a"
  "enhancement" = "a2eeef"
  "question"    = "d876e3"
}
```

> In real projects `terraform.tfvars` is git-ignored — it usually holds secrets and
> per-environment values. Here it only carries label names, so it is safe to keep.

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

First apply what you have and read the output:

```bash
terraform fmt
terraform validate
terraform plan -out lastPlan
terraform apply "lastPlan"
terraform output managed_labels
```

Now change **one thing at a time**. Each experiment below tells you what to edit, the
command to run, and what the plan should say. Always read the plan *before* you apply and
check it matches "Expected".

#### 5.1 — Add a label to the map

In `terraform.tfvars`, add one entry to the `labels` map:

```hcl
labels = {
  "bug"         = "d73a4a"
  "enhancement" = "a2eeef"
  "question"    = "d876e3"
  "docs"        = "0075ca"
}
```

```bash
terraform plan -out lastPlan
terraform apply "lastPlan"
```

Expected: **Plan: 1 to add, 0 to change, 0 to destroy** — only `docs` is created; the
other labels are untouched.

#### 5.2 — Remove a label from the map

In `terraform.tfvars`, delete the `"question"` line. Then:

```bash
terraform plan -out lastPlan
terraform apply "lastPlan"
```

Expected: **Plan: 0 to add, 0 to change, 1 to destroy** — only `question` is destroyed.
With `for_each` every label is keyed by its name, so removing one leaves the rest alone.

#### 5.3 — Turn the conditional label on

```bash
terraform apply -var="enable_release_label=true"
```

Expected: **1 to add** — the `release` label appears (its `count` went from 0 to 1).

#### 5.4 — Turn the conditional label off

```bash
terraform apply -var="enable_release_label=false"
```

Expected: **1 to destroy** — only the `release` label is removed (`count` went 1 to 0).

#### 5.5 — See why for_each beats count for named things

Switch the labels to a positional `count` version to feel the difference. Comment out the
`for_each` `labels` resource from Step 01 and add this instead:

```hcl
variable "label_list" {
  type    = list(string)
  default = ["bug", "enhancement", "question"]
}

resource "github_issue_label" "by_count" {
  count      = length(var.label_list)
  repository = github_repository.course_repo.name
  name       = var.label_list[count.index]
  color      = "ededed"
}
```

Create the three labels:

```bash
terraform apply
```

Now remove the **middle** item (`"enhancement"`) from `label_list` and plan:

```bash
terraform plan
```

Expected: Terraform wants to change `question` too — removing index 1 shifts every later
index, so the resource at index 2 is recreated at index 1. That index-shift is the classic
reason to prefer `for_each` for named resources.

Revert before committing: delete the `by_count` block and the `label_list` variable, and
uncomment the `for_each` `labels` resource.

Commit your work:

```bash
git add . && git commit -m "Lab 04: conditionals and iterations" && git push
```

## Conclusion

One map now drives any number of resources, a boolean toggles a resource on and off, and
an output summarizes the whole collection — all while keeping your code DRY. Next session
you leave GitHub behind and move to Azure.

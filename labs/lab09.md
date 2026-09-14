# Lab 09 - Refactor your project into modules

> Consolidates **Session 09 — Terraform Modules**: module structure, inputs/outputs, and
> refactoring without destroying infrastructure.

## Table of Contents

- [Learning Objectives](#learning-objectives)
- [Pre-requisites](#pre-requisites)
- [Guide](#guide)
  - [Step 01: Plan your modules](#step-01-plan-your-modules)
  - [Step 02: Create the network module (guided)](#step-02-create-the-network-module-guided)
  - [Step 03: Call the module from the root](#step-03-call-the-module-from-the-root)
  - [Step 04: Verify with a no-change plan](#step-04-verify-with-a-no-change-plan)
  - [Step 05: Modularize the app and database (on your own)](#step-05-modularize-the-app-and-database-on-your-own)
  - [Step 06: Commit and push](#step-06-commit-and-push)
- [Conclusion](#conclusion)

## Learning Objectives

- Understand module structure (main / variables / outputs)
- Extract existing resources into reusable modules
- Wire inputs and outputs between the root and child modules
- Refactor **without** destroying and recreating infrastructure

## Pre-requisites

- Have finished [Lab 08](lab08.md) — your Azure infrastructure is applied, state in the backend
- Reference implementation available in [the Session 09 demo](../demos/session09) if you get stuck

> Goal: the infrastructure stays **exactly the same** — you're only reorganizing the
> code. A correct refactor ends with `terraform plan` reporting *no changes*.

## Guide

### Step 01: Plan your modules

Group the resources in your `infra` project by responsibility:

- `network` — resource group + virtual network + subnets
- `app_service` — App Service plan + the two App Services
- `psql` — the PostgreSQL flexible server

```bash
mkdir -p modules/network modules/app_service modules/psql
```

Each module folder gets three files: `main.tf`, `variables.tf`, `outputs.tf`.

### Step 02: Create the network module (guided)

Extract the **network** module together:

1. **Move the resources** — cut the network resource group, virtual network and subnets
   from the root `main.tf` into `modules/network/main.tf`.
2. **Declare inputs** in `modules/network/variables.tf` (`prefix`, `location`, `tags`).
3. **Expose outputs** in `modules/network/outputs.tf` — subnet IDs and the resource group
   name that other modules will need:

   ```hcl
   output "app_subnet_id" { value = azurerm_subnet.app.id }
   output "db_subnet_id"  { value = azurerm_subnet.db.id }
   output "net_rg_name"   { value = azurerm_resource_group.net.name }
   ```
4. **Fix references** inside the module to use `var.` and local resource addresses.

### Step 03: Call the module from the root

In the root `main.tf`, replace the moved resources with a module call:

```hcl
module "network" {
  source   = "./modules/network"
  prefix   = var.prefix
  location = var.location
  tags     = local.common_tags
}
```

Re-initialize so Terraform registers the module:

```bash
terraform init
```

### Step 04: Verify with a no-change plan

Because the resources already exist in state under their old addresses, record the moves
in code with `moved` blocks (or run `terraform state mv`):

```hcl
moved {
  from = azurerm_virtual_network.vnet
  to   = module.network.azurerm_virtual_network.vnet
}
```

Then confirm the goal:

```bash
terraform plan
# Plan: 0 to add, 0 to change, 0 to destroy.
```

A clean, no-change plan means the refactor is correct.

### Step 05: Modularize the app and database (on your own)

Repeat the same pattern **solo** for `app_service` and `psql`, wiring them together with
the network module's outputs:

```hcl
module "app_service" {
  source    = "./modules/app_service"
  prefix    = var.prefix
  location  = var.location
  subnet_id = module.network.app_subnet_id
  tags      = local.common_tags
}
```

Finish again on a **no-change plan**. Compare with [the Session 09 demo](../demos/session09) if needed.

### Step 06: Commit and push

```bash
git add . && git commit -m "Lab 09: refactor infrastructure into modules" && git push
```

## Conclusion

You turned a flat configuration into focused, reusable modules orchestrated by the root —
and proved it with a no-change plan. This is the structure your CI/CD pipeline deploys in
the final session.

# Lab 08 - Import existing resources & manage state

> Consolidates **Session 08 — Importing & Managing State**: bringing brownfield resources
> under Terraform, the `terraform import` command, and detecting drift. You import a Key
> Vault created by hand, then manage a secret and an RBAC assignment.

## Table of Contents

- [Learning Objectives](#learning-objectives)
- [Pre-requisites](#pre-requisites)
- [Guide](#guide)
  - [Step 01: Create the Key Vault manually](#step-01-create-the-key-vault-manually)
  - [Step 02: Import the Key Vault](#step-02-import-the-key-vault)
  - [Step 03: Add a secret](#step-03-add-a-secret)
  - [Step 04: Add the write permission (RBAC)](#step-04-add-the-write-permission-rbac)
  - [Step 05: Detect drift](#step-05-detect-drift)
- [Conclusion](#conclusion)

## Learning Objectives

- Import an existing resource into Terraform state with `terraform import`
- Use the `azurerm_client_config` data source and an `azurerm_role_assignment`
- Use `depends_on` for an explicit dependency
- Detect configuration drift with `terraform plan`

## Pre-requisites

- Have finished [Lab 07](lab07.md) — still working in the `infra` folder

## Guide

### Step 01: Create the Key Vault manually

In the Azure Portal, create a Key Vault named `<your_prefix>-kv` in `westeurope`, in your
resource group from the previous labs. **Enable Azure RBAC** for it. Leave the rest as
default. This simulates a resource that already exists outside Terraform.

### Step 02: Import the Key Vault

Add the matching HCL to `main.tf` so Terraform has something to import into:

```hcl
data "azurerm_client_config" "current" {}

resource "azurerm_key_vault" "kv" {
  name                      = "<your_prefix>-kv"
  location                  = "westeurope"
  resource_group_name       = azurerm_resource_group.rg-app.name
  tenant_id                 = data.azurerm_client_config.current.tenant_id
  sku_name                  = "standard"
  enable_rbac_authorization = true
}
```

Get the Key Vault's Resource ID from the Portal (Settings → Properties). Import it the
**modern, reviewable way** with an `import` block (Terraform 1.5+):

```hcl
import {
  to = azurerm_key_vault.kv
  id = "<the-resource-id>"
}
```

```bash
terraform plan     # shows the resource will be imported, no changes
terraform apply    # performs the import; then remove the import block
```

> The classic CLI equivalent still works: `terraform import azurerm_key_vault.kv "<the-resource-id>"`.
> The `import` block is preferred because it lives in code and shows up in a pull request.

Either way, a follow-up `terraform plan` should report **no changes** — proof your code
matches the real resource. That is a successful import.

### Step 03: Add a secret

```hcl
resource "azurerm_key_vault_secret" "kv_secret" {
  name         = "db-password"
  value        = var.db_password
  key_vault_id = azurerm_key_vault.kv.id
}
```

```bash
terraform plan -out kvSecretPlan
terraform apply kvSecretPlan
```

You'll get a permission error — with RBAC enabled, your user needs a role to write
secrets. That's the next step.

### Step 04: Add the write permission (RBAC)

```hcl
resource "azurerm_role_assignment" "kv_officer" {
  scope                = azurerm_key_vault.kv.id
  role_definition_name = "Key Vault Secrets Officer"
  principal_id         = data.azurerm_client_config.current.object_id
}
```

The secret needs the role to exist first, but there's no implicit link between them — so
add an explicit dependency:

```hcl
resource "azurerm_key_vault_secret" "kv_secret" {
  name         = "db-password"
  value        = var.db_password
  key_vault_id = azurerm_key_vault.kv.id
  depends_on   = [azurerm_role_assignment.kv_officer]
}
```

```bash
terraform plan -out kvSecretPlan
terraform apply kvSecretPlan
```

### Step 05: Detect drift

In the Portal, change a tag on the Key Vault by hand, then:

```bash
terraform plan            # shows the drift as a proposed change
terraform plan -refresh-only   # or reconcile state to match reality
```

This is how idempotence catches configuration drift. Commit your work:

```bash
git add . && git commit -m "Lab 08: import Key Vault, secret and RBAC" && git push
```

## Conclusion

You adopted a hand-created resource into Terraform, managed a secret and an RBAC
assignment with an explicit dependency, and saw how `plan` surfaces drift. Next session
you refactor the whole project into reusable modules.

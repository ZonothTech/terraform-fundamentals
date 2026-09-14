# Lab 05 - State backends & Azure authentication

> Consolidates **Session 05 — State Backends & Azure Auth**: remote state, the `azurerm`
> backend, and authenticating to Azure. This is the bridge from GitHub to Azure — the
> storage account you create here backs the infrastructure you build from Lab 06 on.

## Table of Contents

- [Goals](#goals)
- [Pre-requisites](#pre-requisites)
- [Guide](#guide)
  - [Step 01: Start the Azure project](#step-01-start-the-azure-project)
  - [Step 02: Author the backend storage code](#step-02-author-the-backend-storage-code)
  - [Step 03: Log in to Azure](#step-03-log-in-to-azure)
  - [Step 04: Apply with local state](#step-04-apply-with-local-state)
  - [Step 05: Configure the backend](#step-05-configure-the-backend)
  - [Step 06: Migrate the state](#step-06-migrate-the-state)
- [Conclusion](#conclusion)

## Goals

- Create an Azure Storage account to hold Terraform state
- Configure the `azurerm` backend and migrate state to it
- Authenticate Terraform to Azure with the Azure CLI
- See Azure Blob state locking in action

## Pre-requisites

- Have finished [Lab 04](lab04.md)
- Azure CLI installed (from [Lab 00](lab00-setup.md))
- Your assigned **prefix** and the **shared subscription id** from the instructor

> Shared subscription: prefix every resource with your assigned `<prefix>`, and use a
> per-student state `key` so everyone's state stays isolated in the same subscription.

## Guide

### Step 01: Start the Azure project

Inside your `terraform-training` repo, create a `backend` folder for the storage that
will hold your state. Keep it separate from your app infrastructure.

```bash
mkdir backend && cd backend
```

### Step 02: Author the backend storage code

`versions.tf`:

```hcl
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 4.0"
    }
  }
}

provider "azurerm" {
  subscription_id = "<subscription_id>"
  features {}
}
```

`variables.tf`:

```hcl
variable "prefix" {
  description = "Your assigned prefix (keeps names unique in the shared subscription)."
  type        = string
}

variable "location" {
  type    = string
  default = "westeurope"
}
```

`main.tf`:

```hcl
locals {
  resource_group_name  = "${var.prefix}-terraform-state-rg"
  storage_account_name = "${var.prefix}tfstate"
}

resource "azurerm_resource_group" "rg" {
  name     = local.resource_group_name
  location = var.location
}

resource "azurerm_storage_account" "sa" {
  name                     = local.storage_account_name
  resource_group_name      = azurerm_resource_group.rg.name
  location                 = azurerm_resource_group.rg.location
  account_tier             = "Standard"
  account_replication_type = "LRS"
  min_tls_version          = "TLS1_2"
}

resource "azurerm_storage_container" "sc" {
  name                  = "terraform"
  storage_account_id    = azurerm_storage_account.sa.id
  container_access_type = "private"
}
```

### Step 03: Log in to Azure

```bash
az login --tenant <tenant_id>
az account set --subscription <subscription_id>
```

### Step 04: Apply with local state

```bash
terraform init
terraform plan -var="prefix=<your_prefix>" -out plan
terraform apply "plan"
```

The storage account is created, and its state is still a local `terraform.tfstate`.

### Step 05: Configure the backend

Upload the local state to the container (Azure Portal → your Storage Account → the
`terraform` container → Upload → `terraform.tfstate`), then delete the local
`terraform.tfstate` and `terraform.tfstate.backup`.

Now add a `backend` block inside the `terraform` block in `versions.tf`:

```hcl
backend "azurerm" {
  resource_group_name  = "<your_prefix>-terraform-state-rg"
  storage_account_name = "<your_prefix>tfstate"
  container_name       = "terraform"
  key                  = "<your_prefix>.tfstate"
  use_azuread_auth     = true   # authenticate to the state with Azure AD, not an access key
}
```

> Best practice: `use_azuread_auth = true` uses your Azure AD identity to reach the state
> instead of a storage account access key — one less long-lived secret. In CI/CD you'd
> pair it with `use_oidc = true` (Session 10).

### Step 06: Migrate the state

```bash
terraform init
terraform plan -var="prefix=<your_prefix>" -out plan
```

You should see `No changes` — the same infrastructure, now tracked in Azure. Notice the
log **acquires** and **releases** a state lock: that's Azure Blob lease-based locking
preventing two applies from colliding.

Commit your work:

```bash
git add . && git commit -m "Lab 05: Azure Storage backend for state" && git push
```

## Conclusion

Your state now lives in a locked, shared Azure backend — the foundation every real
project needs. From the next lab on, your Azure infrastructure uses this backend.

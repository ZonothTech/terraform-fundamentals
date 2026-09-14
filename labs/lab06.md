# Lab 06 - Build Azure infrastructure: resource groups & network

> Consolidates **Session 06 — Building Azure Infrastructure**: azurerm patterns, reading
> the provider docs, naming & tags, and networking. You start the application project
> that you'll grow across Labs 06–10.

## Table of Contents

- [Goals](#goals)
- [Pre-requisites](#pre-requisites)
- [Guide](#guide)
  - [Step 01: Start the infrastructure project](#step-01-start-the-infrastructure-project)
  - [Step 02: Create the resource groups](#step-02-create-the-resource-groups)
  - [Step 03: Create the virtual network](#step-03-create-the-virtual-network)
  - [Step 04: Apply and verify](#step-04-apply-and-verify)
- [Conclusion](#conclusion)

## Goals

- Author Azure resources by reading the provider documentation
- Apply a consistent naming (`<prefix>-…`) and tagging convention
- Create three tagged resource groups and a virtual network with subnets

## Pre-requisites

- Have finished [Lab 05](lab05.md) — you have an Azure backend and are logged in with `az`
- Your assigned **prefix**; all resources go in `westeurope`

> On this lab you're given the **properties** — you write the HCL by reading the docs.
> That's the real day-to-day Terraform skill.

## Guide

### Step 01: Start the infrastructure project

In your `terraform-training` repo, create an `infra` folder for the application
infrastructure (separate from the `backend` folder from Lab 05).

```bash
mkdir infra && cd infra
```

Add `versions.tf` with the `azurerm` provider **and** the backend you created in Lab 05
(use a different `key`, e.g. `<prefix>-infra.tfstate`):

```hcl
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 4.0"
    }
  }
  backend "azurerm" {
    resource_group_name  = "<your_prefix>-terraform-state-rg"
    storage_account_name = "<your_prefix>tfstate"
    container_name       = "terraform"
    key                  = "<your_prefix>-infra.tfstate"
  }
}

provider "azurerm" {
  subscription_id = "<subscription_id>"
  features {}
}
```

Add a `variables.tf` with `prefix` and `location`, and a `locals` block with common tags:

```hcl
locals {
  common_tags = {
    environment = "prod"
    owner       = var.prefix
  }
}
```

### Step 02: Create the resource groups

Create three resource groups in `westeurope`, each with the common tags plus a `type`
tag. Ignore manual changes to tags (`lifecycle { ignore_changes = [tags] }`).

- `<your_prefix>-net-rg` → `type = "network"`
- `<your_prefix>-app-rg` → `type = "application"`
- `<your_prefix>-db-rg`  → `type = "database"`

Docs: [azurerm_resource_group](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/resource_group).

### Step 03: Create the virtual network

Create a virtual network in the `net-rg` with two subnets:

- Name: `<your_prefix>-vnet`, address space `10.20.30.0/24`
- Subnet `app`: `10.20.30.0/27`
- Subnet `db`: `10.20.30.32/27`

Tag it with the same common tags. Docs:
[azurerm_virtual_network](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/virtual_network).

### Step 04: Apply and verify

```bash
terraform fmt
terraform init
terraform validate
terraform plan -var="prefix=<your_prefix>" -out infraPlan
terraform apply infraPlan
```

Check the resource groups and virtual network in the Azure Portal, then commit and push:

```bash
git add . && git commit -m "Lab 06: resource groups and virtual network" && git push
```

> Real Azure resources cost money — read every plan, and `terraform destroy` what you
> don't need between sessions.

## Conclusion

You built the networking foundation of your app straight from the provider docs, with a
clean naming and tagging convention. Next session you add the application tier.

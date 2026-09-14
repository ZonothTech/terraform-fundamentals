# Lab 07 - Advanced resources: App Services & PostgreSQL

> Consolidates **Session 07 — Advanced Resources**: data sources, explicit dependencies,
> dynamic blocks and the resource lifecycle. You finish the application tier of your
> Azure project.

## Table of Contents

- [Goals](#goals)
- [Pre-requisites](#pre-requisites)
- [Guide](#guide)
  - [Step 01: Add a data source](#step-01-add-a-data-source)
  - [Step 02: Create the App Service plan and App Services](#step-02-create-the-app-service-plan-and-app-services)
  - [Step 03: Create the PostgreSQL database](#step-03-create-the-postgresql-database)
  - [Step 04: Use lifecycle to ignore tag drift](#step-04-use-lifecycle-to-ignore-tag-drift)
  - [Step 05: Apply and verify](#step-05-apply-and-verify)
- [Conclusion](#conclusion)

## Goals

- Query existing information with a data source
- Create App Services on an App Service plan, integrated with the VNet
- Create a managed PostgreSQL database
- Control resource behavior with the `lifecycle` block

## Pre-requisites

- Have finished [Lab 06](lab06.md) — resource groups and virtual network are applied
- Still working in the `infra` folder, logged in with `az`

## Guide

### Step 01: Add a data source

Data sources **read** existing information at plan time. Add the current client config —
you'll reuse this pattern in Lab 08:

```hcl
data "azurerm_client_config" "current" {}
```

You can reference the subnets you created in Lab 06 directly
(`azurerm_subnet.app.id`) to integrate the App Services with the network.

### Step 02: Create the App Service plan and App Services

Create an App Service plan and two Linux App Services in the `app-rg`:

- Plan: Basic `B1`, Linux
- `<your_prefix>-front-app` and `<your_prefix>-api-app`, runtime `.NET 8`
- Set `https_only = true` on both (a Checkov check you'll want to pass in Session 10)
- Both integrated with the `app` subnet of your VNet
- Tag everything with your common tags

Docs:
[azurerm_service_plan](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/service_plan) ·
[azurerm_linux_web_app](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/linux_web_app).

### Step 03: Create the PostgreSQL database

Create a PostgreSQL Flexible Server in the `db-rg`:

- Name `<your_prefix>-psql`, SKU `B_Standard_B1ms`, version `16`, storage `32768`
- Set an admin username, and an admin password from a **variable marked `sensitive`**

```hcl
variable "db_password" {
  type      = string
  sensitive = true
}
```

Docs:
[azurerm_postgresql_flexible_server](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/postgresql_flexible_server).

### Step 04: Use lifecycle to ignore tag drift

External processes sometimes edit tags. Tell Terraform to stop fighting them by adding a
`lifecycle` block to your resources:

```hcl
lifecycle {
  ignore_changes = [tags]
}
```

This is exactly what "ignore manual changes to tags" means in practice. Other lifecycle
options worth knowing: `create_before_destroy` (zero-downtime replacement) and
`prevent_destroy` (guard costly resources).

### Step 05: Apply and verify

```bash
terraform fmt
terraform validate
terraform plan -var="prefix=<your_prefix>" -var="db_password=<a-strong-password>" -out infraPlan
terraform apply infraPlan
```

Verify the App Services and database in the Azure Portal, then commit and push:

```bash
git add . && git commit -m "Lab 07: App Services and PostgreSQL" && git push
```

## Conclusion

Your application tier is live — App Services and a database — built with data sources and
lifecycle rules that make the code robust against real-world drift. Next session you
bring an externally-created resource under Terraform's control.

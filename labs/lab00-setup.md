# Lab 00 - Course setup (do this BEFORE Session 01)

Complete this short setup **before** the first session so we can spend class time on
Terraform, not installations. It takes about 15–20 minutes.

## Table of Contents

- [What you'll need](#what-youll-need)
- [Step 01: Install a code editor](#step-01-install-a-code-editor)
- [Step 02: Install Terraform](#step-02-install-terraform)
- [Step 03: Install the GitHub CLI](#step-03-install-the-github-cli)
- [Step 04: Install the Azure CLI](#step-04-install-the-azure-cli)
- [Step 05: Accounts & the shared Azure subscription](#step-05-accounts--the-shared-azure-subscription)
- [Step 06: Verify everything](#step-06-verify-everything)

## What you'll need

- A computer with a terminal/console
- A GitHub account (a personal account is fine)
- Access to the **shared Azure subscription** and your assigned **prefix** (provided by the instructor)

> All lab guides use macOS/Linux commands. On Windows we recommend
> [WSL](https://docs.microsoft.com/en-us/windows/wsl/install) for the best experience.

## Step 01: Install a code editor

We recommend [Visual Studio Code](https://code.visualstudio.com/) with the
[HashiCorp Terraform extension](https://marketplace.visualstudio.com/items?itemName=HashiCorp.terraform)
— it gives you syntax highlighting, validation and inline errors.

## Step 02: Install Terraform

Follow the [official install guide](https://developer.hashicorp.com/terraform/install). Then check:

```bash
terraform --version
# Terraform v1.9.x (or newer)
```

## Step 03: Install the GitHub CLI

Follow the [GitHub CLI install guide](https://cli.github.com/). Then check:

```bash
gh --version
```

## Step 04: Install the Azure CLI

Needed from Session 05 onwards. Follow the
[Azure CLI install guide](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli). Then check:

```bash
az --version
```

## Step 05: Accounts & the shared Azure subscription

- Sign in to GitHub in your browser.
- The course uses **one shared Azure subscription**. To avoid name clashes, every
  student gets a unique **prefix** (for example `tb01`, `tb02`, …). You will use it
  everywhere resources are named: `<prefix>-net-rg`, `<prefix>-vnet`, and so on.
- You have **Contributor** rights scoped to your own resource groups, plus access to
  one shared storage account for Terraform state (each student uses a distinct state
  `key`). The instructor will share the subscription id and your prefix.
- All resources go in the **westeurope** region unless stated otherwise.

> Keep your prefix handy — you'll set it as a variable value (`prefix = "<prefix>"`)
> or as `TF_VAR_prefix` in most labs.

## Step 06: Verify everything

```bash
terraform --version   # 1.9+
gh --version
az --version
gh auth status || echo "you'll log in during Lab 01"
```

If all three tools report a version, you're ready for Session 01. See you there!

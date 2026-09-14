<div align="center">

<img src="decks/assets/zonoth-logo.svg" alt="Zonoth" height="48">

# Terraform for Cloud

## A hands-on, project-based Terraform course for Azure — from your first `terraform apply` to a full CI/CD pipeline.

### 20 hours · 10 live sessions of 2h · for engineers with zero Terraform experience

![Duration](https://img.shields.io/badge/duration-20h-3ce8a6?style=flat-square&labelColor=12242a)
![Sessions](https://img.shields.io/badge/sessions-10%20%C3%97%202h-3ce8a6?style=flat-square&labelColor=12242a)
![Cloud](https://img.shields.io/badge/cloud-Azure-12a85c?style=flat-square&labelColor=12242a)
![Providers](https://img.shields.io/badge/providers-azurerm%20%2B%20GitHub-12a85c?style=flat-square&labelColor=12242a)
![Level](https://img.shields.io/badge/level-beginner%20%E2%86%92%20practitioner-12a85c?style=flat-square&labelColor=12242a)

[**Slide decks**](./decks/index.html) · [**Setup guide**](./labs/lab00-setup.md) · [**Curriculum**](#curriculum) · [**Labs**](#labs)

</div>

---

## Why this course

Most Terraform material stops at "here's a resource block." This one is built around a **single project you carry from session one to session ten**: Terraform creates your Git repository on day one, and from there you build, secure, import, modularize and continuously deliver a real Azure web application — network, App Services and a PostgreSQL database — all as code, in a repository Terraform itself created.

You learn the way you'll actually work: read the plan, commit the change, open the pull request, ship it.

## What you'll build

> **The storyline.** You join a platform team. Terraform provisions your Git repo (GitHub provider), then you author the infrastructure for an Azure web app, move its state to a locked cloud backend, import a resource created by hand, refactor everything into reusable modules, and finally gate it all behind a pull-request pipeline with security scanning.

Every session ends in a **lab** on that same project, so nothing is throwaway — the thing you finish the course with is a complete, modular, CI/CD-delivered Azure environment in your own repository.

## Who it's for

- **Software engineers** who want their infrastructure to live in code and pull requests, not portals.
- **Infra / SysAdmin / DevOps** practitioners moving from click-ops to Infrastructure as Code.
- Anyone technical and comfortable in a terminal — **no prior Terraform experience needed**.

## What you'll be able to do by the end

- Author, plan and apply Terraform confidently, and read state like a map of your infra.
- Model variables, locals, outputs, conditionals and iteration to keep configurations DRY.
- Store state safely in an Azure Storage backend and authenticate the right way for local and CI/CD.
- Build Azure infrastructure from the provider docs — networking, App Services, PostgreSQL.
- Import existing ("brownfield") resources and detect configuration drift.
- Package infrastructure into reusable modules.
- Scan and ship changes through a GitHub Actions pull-request and CI/CD workflow.

---

## Curriculum

Each 2-hour session runs the same rhythm: **50 min theory + demos · 10 min break · 20–30 min theory + demos · 30–40 min lab.**

| # | Session | You'll learn | Deck | Lab |
|---|---------|--------------|------|-----|
| 01 | **Introduction to IaC & Terraform** | What IaC is, HCL, providers, the core workflow | [deck](./decks/session-01.pdf) | [Lab 01](./labs/lab01.md) |
| 02 | **Main Concepts** | CLI, providers, resources, dependencies, state | [deck](./decks/session-02.pdf) | [Lab 02](./labs/lab02.md) |
| 03 | **Variables & Outputs** | Inputs, precedence, validation, locals, outputs | [deck](./decks/session-03.pdf) | [Lab 03](./labs/lab03.md) |
| 04 | **Conditionals & Iterations** | `count`, `for_each`, `for` expressions | [deck](./decks/session-04.pdf) | [Lab 04](./labs/lab04.md) |
| 05 | **State Backends & Azure Auth** | Remote state, azurerm backend, Azure authentication | [deck](./decks/session-05.pdf) | [Lab 05](./labs/lab05.md) |
| 06 | **Building Azure Infrastructure** | azurerm patterns, reading docs, naming & tags, networking | [deck](./decks/session-06.pdf) | [Lab 06](./labs/lab06.md) |
| 07 | **Advanced Resources** | Data sources, `depends_on`, dynamic blocks, lifecycle | [deck](./decks/session-07.pdf) | [Lab 07](./labs/lab07.md) |
| 08 | **Importing & Managing State** | Brownfield import, state surgery, drift | [deck](./decks/session-08.pdf) | [Lab 08](./labs/lab08.md) |
| 09 | **Terraform Modules** | Structure, registry, private sharing, versioning | [deck](./decks/session-09.pdf) | [Lab 09](./labs/lab09.md) |
| 10 | **Terraform in CI/CD** | Checkov, PR workflow, disposable envs, drift | [deck](./decks/session-10.pdf) | [Lab 10](./labs/lab10.md) |

Prefer PDFs? The slides also export to PDF (see [Slide decks](#slide-decks)).

---

## Getting started

1. **Before session 1**, complete the short [**setup guide**](./labs/lab00-setup.md) — install Terraform, the GitHub CLI, the Azure CLI and a code editor, and note your shared-subscription prefix. ~15 minutes.
2. Work each session's **lab** in the same `terraform-training` project — it's the through-line of the whole course.

## Repository structure

```
├── decks/            PDF slide decks
├── labs/             Step-by-step hands-on labs (lab00 setup → lab10 CI/CD)
```

## Labs

Every session ends in a hands-on lab, and they all build on the **same `terraform-training` project** — so by the end you have one complete, modular, CI/CD-delivered Azure environment in your own repo. Start with the setup guide before session one.

| Lab | Session | What you'll do | Provider |
|-----|:-------:|----------------|----------|
| [Lab 00 — Course setup](./labs/lab00-setup.md) | pre-work | Install Terraform, the GitHub CLI, the Azure CLI and an editor; get your shared-subscription prefix. ~15 min. | — |
| [Lab 01 — Your first commands](./labs/lab01.md) | 01 | Author your first HCL and use Terraform to create the Git repo that hosts your project all course long — `init` → `plan` → `apply` → read state → `destroy`. | GitHub |
| [Lab 02 — Organize code, resources & state](./labs/lab02.md) | 02 | Split into `main`/`versions`/`outputs`, add a second resource to see dependency ordering, and read state &amp; the lock file. | GitHub |
| [Lab 03 — Variables, locals & outputs](./labs/lab03.md) | 03 | Add typed &amp; validated variables, tfvars and precedence, locals, and sensitive outputs. | GitHub |
| [Lab 04 — Conditionals & iterations](./labs/lab04.md) | 04 | Drive resources from a map with `for_each`, toggle one with `count`, and build an output over a collection. | GitHub |
| [Lab 05 — State backends & Azure auth](./labs/lab05.md) | 05 | Create an Azure Storage account and migrate your state to a locked remote `azurerm` backend. | azurerm |
| [Lab 06 — Azure infra: groups & network](./labs/lab06.md) | 06 | Docs-driven: build three tagged resource groups and a virtual network with subnets. | azurerm |
| [Lab 07 — Advanced resources: apps & DB](./labs/lab07.md) | 07 | Add App Services and PostgreSQL using data sources and `lifecycle` (ignore tag drift). | azurerm |
| [Lab 08 — Import & manage state](./labs/lab08.md) | 08 | Create a Key Vault by hand, import it, then manage a secret and an RBAC assignment with `depends_on`. | azurerm |
| [Lab 09 — Refactor into modules](./labs/lab09.md) | 09 | Guided refactor: extract a `network` module together, then modularize app and database — ending on a no-change plan. | azurerm |
| [Lab 10 — Terraform in CI/CD](./labs/lab10.md) | 10 | Branch protection and secretless OIDC, a Checkov + plan pull-request workflow, then the CD finale — disposable env, approvals and drift. | GitHub Actions |

---

## Feedback

Found something to improve? Open an issue describing what you hit and I'll get back to you.

<div align="center">

**Zonoth** • Terraform for Cloud

</div>

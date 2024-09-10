---
title: Terraform - Infrastructure as Code and the State File
description: Terraform builds cloud infrastructure from a declarative description and a record of what it already built. How the provider model, the resource graph, and the state file fit together, why state is the source of most Terraform pain, and how it compares with Pulumi, CloudFormation, and Ansible.
date: 2024-09-10
draft: false
slug: /tech/terraform
tags:
  - Tools
  - DevOps
  - Cloud
---

Clicking through a cloud console to create a VPC, three subnets, a load balancer, an autoscaling group, and the security groups that tie them together works once. Then you need the same thing in a second region, or a teammate needs to reproduce it, or an audit asks what changed last quarter — and clicking doesn't answer any of that.

**Infrastructure as code** replaces the clicking with a text file you commit to git. **Terraform** is the most widely used tool for it: you describe the infrastructure you want, Terraform figures out the difference between that and what exists, and makes the minimum set of API calls to close the gap. This post is the three things that make Terraform tick — providers, the resource graph, and the state file — and where the state file will bite you.

## Declarative, not a script

You don't write "create this, then create that." You write *what should exist*:

```hcl
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
  tags       = { Name = "prod" }
}

resource "aws_subnet" "app" {
  vpc_id            = aws_vpc.main.id      # reference creates a dependency
  cidr_block        = "10.0.1.0/24"
  availability_zone = "us-east-1a"
}
```

Terraform reads all the `.tf` files in a directory, builds a graph of the resources and their dependencies (the `aws_subnet` references `aws_vpc.main.id`, so the VPC must exist first), and walks that graph. Independent resources are created in parallel. You never specify order; the references imply it.

The workflow is three commands:

- **`terraform init`** — download the providers this configuration uses and set up the backend.
- **`terraform plan`** — compute the diff between your config, the state, and reality, and print exactly what it would add, change, or destroy. Read this every time.
- **`terraform apply`** — execute that plan.

## Providers

A **provider** is a plugin that translates Terraform's generic resource operations into a specific API's calls. `hashicorp/aws`, `hashicorp/google`, `hashicorp/azurerm`, plus hundreds more — Cloudflare, Datadog, GitHub, PostgreSQL, Kubernetes. Each provider defines the resource types (`aws_instance`, `google_storage_bucket`) and their schemas.

Because it's just a provider, Terraform can manage things that aren't "cloud infrastructure" at all: DNS records, SaaS configuration, database roles, PagerDuty schedules. Anything with a CRUD API can have a provider.

Pin provider versions in a `required_providers` block. A provider is software; a minor version bump can change a default or a validation and turn your next `plan` into a surprise.

## The state file

This is the part that's different from a normal build tool, and the source of most Terraform incidents.

Terraform keeps a **state file** (`terraform.tfstate`, JSON) mapping each resource in your config to the real object it created — the AWS instance ID, the current attribute values, the dependency edges. `plan` works by comparing three things: your configuration, the state, and (via a **refresh**) the live infrastructure.

Why not just query the cloud every time and skip the state? Because:

- **Identity.** Your config says `resource "aws_instance" "web"`. The cloud has an instance `i-0abc123`. Only the state knows those are the same thing. Without it, Terraform can't tell "update this instance" from "the instance is gone, make a new one."
- **Performance.** Refreshing hundreds of resources against a live API on every command is slow; state caches the last-known values.
- **Metadata.** Dependency order and provider-specific data that isn't readable back from the API.

Consequences you have to design around:

- **State must be shared and locked.** If two engineers `apply` against the same infra with local state files, they clobber each other. Use a **remote backend** — S3 + DynamoDB lock table, Terraform Cloud, GCS — so state lives in one place and `apply` takes a lock.
- **State contains secrets.** Any resource attribute that's sensitive (a generated DB password, a private key) is written to state in plaintext. Encrypt the backend; restrict who can read it.
- **`plan` shows drift.** If someone changes infrastructure by hand, the next refresh notices, and `plan` will offer to revert it. That's the feature working — but it means out-of-band changes are hostile to Terraform.
- **Never hand-edit state.** To restructure without destroying resources, use `terraform state mv` (rename), `terraform import` (adopt an existing object), `removed` blocks (forget without destroying), and `moved` blocks (refactor module paths). Editing the JSON directly is how you end up with Terraform trying to delete production.

## Modules and environments

A **module** is a directory of `.tf` files you can call with inputs, like a function. Factor a reusable unit — "a service: its ECS task, ALB target group, security group, and DNS record" — into a module, version it in its own repo or a registry, and instantiate it per service.

For multiple environments (dev, staging, prod), the common patterns are:

- **Directory per environment**, each with its own backend/state, all calling shared modules. Explicit, a little repetitive, hard to apply the wrong thing to the wrong environment. Usually the right choice.
- **Workspaces** — one config, multiple named states. Lighter, but easy to run `apply` in the wrong workspace, and it hides the environment differences in variable files.

## How it compares

| Tool | Model | Language | State | Scope |
|---|---|---|---|---|
| **Terraform** | Declarative | HCL | Explicit file | Any provider |
| **Pulumi** | Declarative | Python/TS/Go/C# | Explicit (managed service or self-hosted) | Any provider (uses TF providers) |
| **CloudFormation / CDK** | Declarative | YAML / real language | Managed by AWS | AWS only |
| **Ansible** | Procedural (mostly) | YAML | Stateless (queries each run) | Config management + some provisioning |

- **Pulumi** is Terraform's model in a general-purpose language, so you get loops, conditionals, and unit tests without HCL's `count`/`for_each` gymnastics. The trade is that "infrastructure that's also a program" can grow logic that's hard to review.
- **CloudFormation** needs no state file (AWS tracks it) and no separate tool to install, but it's AWS-only and historically slower to support new services than the Terraform AWS provider.
- **Ansible** shines at *configuring* servers (install packages, template files, restart services) and is fine for light provisioning, but it has no dependency graph or plan/diff for infrastructure and re-derives state every run.

Many teams use Terraform for the cloud substrate and Ansible or a container image for what runs on top.

## Testing and CI

- `terraform fmt` and `terraform validate` as pre-commit hooks.
- `tflint` for provider-specific lint; `checkov` / `tfsec` / OPA-Conftest for policy ("no public S3 buckets").
- `terraform plan` in CI on every PR, posted as a comment, so reviewers see the infrastructure diff alongside the code diff.
- `apply` gated behind merge to main, often via **Atlantis** or Terraform Cloud so the run is logged and locked rather than happening on someone's laptop.

## The one idea to keep

Terraform is a diff engine: it compares your declarative config, a state file recording what it built, and the live cloud, then makes the minimum API calls to reconcile them. The state file is what gives it identity and speed — and it's also shared mutable state that holds secrets and must be locked, so treat the backend as production infrastructure in its own right, and change resource structure with `state mv` / `import` / `moved`, never by editing the JSON.

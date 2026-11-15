---
name: terraform-module
description: Use when creating a new Terraform module from scratch or refactoring an existing one to match this project's conventions. Covers file layout, variable patterns, output design, README generation, and the validation checklist.
context: fork
---

# Create a Terraform module

This skill walks through creating a new module that matches the conventions defined in `.claude/rules/terraform.md`.

## Step 1 — Clarify scope

Before writing anything, confirm with the user:

1. What does this module manage? (one logical thing — "an S3 bucket with logging", not "the whole data platform")
2. What are the required inputs vs. the optional ones?
3. What outputs do consumers need? (Be conservative — adding outputs later is easy; removing them is breaking.)
4. Is there an existing similar module to use as a reference?

## Step 2 — Scaffold the files

Create the directory structure described in `.claude/rules/terraform.md`. Empty files are fine for now:

```bash
mkdir -p modules/<name>/examples/basic
cd modules/<name>
touch main.tf variables.tf outputs.tf versions.tf README.md
```

## Step 3 — Write `versions.tf` first

This pins the contract. Example:

```hcl
terraform {
  required_version = ">= 1.6"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}
```

## Step 4 — Write `variables.tf`

Every variable gets `description`, `type`, and (where applicable) `validation` and `default`.

## Step 5 — Write `main.tf`

Resources only. No `locals` here — those go in `locals.tf` if needed.

## Step 6 — Write `outputs.tf`

One output per logical concept.

## Step 7 — Write the example

A working `examples/basic/main.tf` that consumes the module with minimal inputs. This is the de-facto integration test.

## Step 8 — Validate

Run the checklist from `.claude/rules/terraform.md`:

```bash
terraform fmt -recursive
terraform validate
tflint
terraform-docs markdown table --output-file README.md .
```

## Step 9 — Hand back to the user

Summarize what was created, with file paths. Do not run `terraform plan` against any real state — that's the user's call.

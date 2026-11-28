# Google Cloud: Inventory — Baseline Setup for School Lab (Project 016001)

## Project overview

Shared classroom equipment provides the setting for this inventory exercise. The embedded code is a reference implementation based on the student guide and uses fictional data.

## Problem statement

The school lab needs a repeatable inventory baseline that another student can verify.

## Project objectives

- Record the school lab inventory baseline.
- Declare a labeled Cloud Storage bucket with versioning and public access prevention.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Terraform HCL and Google Cloud Storage.

## System architecture

`Inventory parameters → infrastructure template → local validation → declared asset baseline`

## Features

- Record the school lab inventory baseline.
- Declare a labeled Cloud Storage bucket with versioning and public access prevention.

## Prerequisites

Use Terraform with the Google provider for a provider-schema check. Use a text editor and a local practice directory.

## Complete Google Cloud source code

```hcl
terraform {
  required_providers {
    google = {
      source  = "hashicorp/google"
      version = "~> 6.0"
    }
  }
}

variable "project_id" {
  type        = string
  description = "ID of an authorized practice Google Cloud project"
}

variable "bucket_name" {
  type        = string
  description = "Globally unique name for the lab inventory bucket"
}

provider "google" {
  project = var.project_id
}

resource "google_storage_bucket" "lab_inventory" {
  name                        = var.bucket_name
  location                    = "US"
  uniform_bucket_level_access = true
  public_access_prevention    = "enforced"

  versioning {
    enabled = true
  }

  labels = {
    asset       = "lab-17"
    environment = "practice"
  }
}
```

## Implementation explanation

Declare a labeled Cloud Storage bucket with versioning and public access prevention.

## Sample input data

Use this example input.

```json
{
  "project_id": "example-practice-project",
  "bucket_name": "example-lab-inventory-17"
}
```

## Execution instructions

Save the source as `main.tf`, then run `terraform init -backend=false` followed by `terraform validate`. Provider initialization requires network access. These commands are for local validation and neither was executed for the portfolio.

## Example output

Illustrative excerpt for the synthetic sample.

```json
{
  "resource_type": "google_storage_bucket",
  "asset_label": "lab-17",
  "public_access_prevention": "enforced",
  "uniform_bucket_level_access": true,
  "versioning": true
}
```

## Testing and validation

A local HCL parser accepted the configuration. Assertions inspected the resource block and verified the inventory label and settings shown below.

```json
{
  "resource_type": "google_storage_bucket",
  "asset_label": "lab-17",
  "public_access_prevention": "enforced",
  "uniform_bucket_level_access": true,
  "versioning": true
}
```

## Error handling

Check syntax, resource types, and input names before deployment. Always use an authorized practice resource group or project for provider-side validation.

## Troubleshooting

- Confirm the documented runtime and copy the complete source block.
- Start with the documented sample and use the filename or request path shown above.
- Compare the observed value with the example before changing an input.

## Limitations

This example covers the declared configuration. No cloud resources were provisioned or checked in a live account.

## Learning outcomes

Practice inventory using Google Cloud and check a repeatable baseline setup workflow in the school lab scenario.

## Conclusion

The project records an observable inventory result for training case `LAB-17`.

[Back to the Google Cloud project index](../README.md)

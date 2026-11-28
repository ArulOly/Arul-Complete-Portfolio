# Terraform: Service Check — Reporting for Two-Node Lab (Project 046639)

## Project overview

A two-node lab simulation provides the setting for this service check exercise. This project includes source code, sample data, and instructions to run it. The readings are synthetic.

## Problem statement

A two-node lab simulation needs a repeatable check for healthy service checks.

## Project objectives

- Review healthy service checks and flag readings that meet the cutoff rule.
- Return the detailed readings and flag count for review.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Terraform HCL, local values, input validation, and outputs.

## System architecture

`Synthetic observations → configuration → native evaluation → report`

## Features

- Review healthy service checks and flag readings that meet the cutoff rule.
- Return the detailed readings and flag count for review.

## Prerequisites

Use Terraform 1.4 or later. Use a text editor and a local practice directory.

## Complete Terraform source code

```hcl
terraform {
  required_version = ">= 1.4"
}

variable "primary" {
  type    = list(number)
  default = [2, 4]
  validation {
    condition     = alltrue([for n in var.primary : n >= 0 && n <= 1000000000 && floor(n) == n])
    error_message = "Use nonnegative whole readings."
  }
}

variable "fallback" {
  type    = list(number)
  default = []
  validation {
    condition     = alltrue([for n in var.fallback : n >= 0 && n <= 1000000000 && floor(n) == n])
    error_message = "Use nonnegative whole fallback readings."
  }
}

locals {
  mode       = "report"
  metric     = "healthy service checks"
  threshold  = 3
  comparison = "lt"
  source     = length(var.primary) > 0 ? "primary" : "fallback"
  readings   = local.source == "primary" ? var.primary : local.mode == "recovery" ? var.fallback : []
  flagged = [for reading in local.readings : reading if (
    local.comparison == "lt" ? reading < local.threshold :
    local.comparison == "gt" ? reading > local.threshold : reading >= local.threshold
  )]
  summary = {
    source   = local.source
    readings = length(local.readings)
    flagged  = length(local.flagged)
  }
}

output "report" {
  description = "Plan-time findings for healthy service checks"
  value = merge(local.summary, contains(["baseline", "report"], local.mode) ? {
    values = local.readings
  } : {})
  precondition {
    condition     = length(local.readings) > 0
    error_message = "Provide readings for the selected input."
  }
}
```

## Implementation explanation

The comparison is lt with limit 3. A flagged row means health checks failed. Return the detailed readings and flag count for review.

## Sample input data

The example embeds these synthetic inputs.

```json
{
  "project": "046639",
  "asset": "NODE-B",
  "metric": "healthy service checks",
  "limit": 3,
  "comparison": "lt",
  "finding": "health checks failed",
  "mode": "report",
  "primary": [
    2,
    4
  ],
  "fallback": []
}
```

## Execution instructions

Save the block as `main.tf` in an empty directory and run `terraform plan -input=false` to inspect the report output. The configuration requires no provider or backend initialization.

## Example output

Illustrative excerpt for the synthetic sample.

```json
{
  "source": "primary",
  "readings": 2,
  "flagged": 1,
  "values": [
    2,
    4
  ]
}
```

## Testing and validation

Ran Terraform plan and inspected the report with Terraform show. The plan contained no resource changes. A negative-reading variant in the batch failed variable validation.

```json
{
  "flagged": 1,
  "readings": 2,
  "source": "primary",
  "values": [
    2,
    4
  ]
}
```

## Error handling

The implementation rejects invalid numeric inputs. Recovery uses the fallback only when the primary list is empty.

## Troubleshooting

- Confirm the documented runtime and copy the complete source block.
- Start with the documented sample and use the filename or request path shown above.
- Compare the observed value with the example before changing an input.

## Limitations

Terraform computes an output from synthetic inputs. The configuration declares no cloud resources.

## Learning outcomes

Practice service check using Terraform and check a repeatable reporting workflow in the two-node lab scenario.

## Conclusion

The project records an observable service check result for training case `NODE-B`.

[Back to the Terraform project index](../README.md)

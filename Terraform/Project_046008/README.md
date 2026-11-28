# Terraform: Inventory — Recovery for Small Business (Project 046008)

## Project overview

Local shop operations provides the setting for this inventory exercise. This project includes source code, sample data, and instructions to run it. The readings are synthetic.

## Problem statement

Local shop operations needs a repeatable check for units in stock.

## Project objectives

- Review units in stock and flag readings that meet the cutoff rule.
- Use the fallback observations when the primary set is empty.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Terraform HCL, local values, input validation, and outputs.

## System architecture

`Synthetic observations → configuration → native evaluation → report`

## Features

- Review units in stock and flag readings that meet the cutoff rule.
- Use the fallback observations when the primary set is empty.

## Prerequisites

Use Terraform 1.4 or later. Use a text editor and a local practice directory.

## Complete Terraform source code

```hcl
terraform {
  required_version = ">= 1.4"
}

variable "primary" {
  type    = list(number)
  default = []
  validation {
    condition     = alltrue([for n in var.primary : n >= 0 && n <= 1000000000 && floor(n) == n])
    error_message = "Use nonnegative whole readings."
  }
}

variable "fallback" {
  type    = list(number)
  default = [2, 5]
  validation {
    condition     = alltrue([for n in var.fallback : n >= 0 && n <= 1000000000 && floor(n) == n])
    error_message = "Use nonnegative whole fallback readings."
  }
}

locals {
  mode       = "recovery"
  metric     = "units in stock"
  threshold  = 4
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
  description = "Plan-time findings for units in stock"
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

The comparison is lt with limit 4. A flagged row means reorder required. Use the fallback observations when the primary set is empty.

## Sample input data

The example embeds these synthetic inputs.

```json
{
  "project": "046008",
  "asset": "SHOP-22",
  "metric": "units in stock",
  "limit": 4,
  "comparison": "lt",
  "finding": "reorder required",
  "mode": "recovery",
  "primary": [],
  "fallback": [
    2,
    5
  ]
}
```

## Execution instructions

Save the block as `main.tf` in an empty directory and run `terraform plan -input=false` to inspect the report output. The configuration requires no provider or backend initialization.

## Example output

Illustrative excerpt for the synthetic sample.

```json
{
  "source": "fallback",
  "readings": 2,
  "flagged": 1
}
```

## Testing and validation

Ran Terraform plan and inspected the report with Terraform show. The plan contained no resource changes. A negative-reading variant in the batch failed variable validation.

```json
{
  "flagged": 1,
  "readings": 2,
  "source": "fallback"
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

Practice inventory using Terraform and check a repeatable recovery workflow in the small business scenario.

## Conclusion

The project records an observable inventory result for training case `SHOP-22`.

[Back to the Terraform project index](../README.md)

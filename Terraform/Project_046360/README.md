# Terraform: Dashboard — Usability Improvement for Small Business (Project 046360)

## Project overview

Local shop operations provides the setting for this dashboard exercise. This project includes source code, sample data, and instructions to run it. The readings are synthetic.

## Problem statement

Local shop operations needs a repeatable check for active tickets.

## Project objectives

- Review active tickets and flag readings that meet the cutoff rule.
- Use a concise result that labels the metric and its flagged count.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Terraform HCL, local values, input validation, and outputs.

## System architecture

`Synthetic observations → configuration → native evaluation → report`

## Features

- Review active tickets and flag readings that meet the cutoff rule.
- Use a concise result that labels the metric and its flagged count.

## Prerequisites

Use Terraform 1.4 or later. Use a text editor and a local practice directory.

## Complete Terraform source code

```hcl
terraform {
  required_version = ">= 1.4"
}

variable "primary" {
  type    = list(number)
  default = [18]
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
  mode       = "summary"
  metric     = "active tickets"
  threshold  = 12
  comparison = "gt"
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
  description = "Plan-time findings for active tickets"
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

The comparison is gt with limit 12. A flagged row means queue above capacity. Use a concise result that labels the metric and its flagged count.

## Sample input data

The example embeds these synthetic inputs.

```json
{
  "project": "046360",
  "asset": "SHOP-22",
  "metric": "active tickets",
  "limit": 12,
  "comparison": "gt",
  "finding": "queue above capacity",
  "mode": "summary",
  "primary": [
    18
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
  "readings": 1,
  "flagged": 1
}
```

## Testing and validation

Ran Terraform plan and inspected the report with Terraform show. The plan contained no resource changes. A negative-reading variant in the batch failed variable validation.

```json
{
  "flagged": 1,
  "readings": 1,
  "source": "primary"
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

Practice dashboard using Terraform and check a repeatable usability improvement workflow in the small business scenario.

## Conclusion

The project records an observable dashboard result for training case `SHOP-22`.

[Back to the Terraform project index](../README.md)

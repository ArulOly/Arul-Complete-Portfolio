# Terraform: Scheduled Reporting — Reporting for School Lab (Project 046454)

## Project overview

Shared classroom equipment provides the setting for this scheduled reporting exercise. This project includes source code, sample data, and instructions to run it. The readings are synthetic.

## Problem statement

Shared classroom equipment needs a repeatable check for days since last report.

## Project objectives

- Review days since last report and flag readings that meet the cutoff rule.
- Return the detailed readings and flag count for review.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Terraform HCL, local values, input validation, and outputs.

## System architecture

`Synthetic observations → configuration → native evaluation → report`

## Features

- Review days since last report and flag readings that meet the cutoff rule.
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
  default = [8, 6]
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
  metric     = "days since last report"
  threshold  = 7
  comparison = "ge"
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
  description = "Plan-time findings for days since last report"
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

The comparison is ge with limit 7. A flagged row means report due. Return the detailed readings and flag count for review.

## Sample input data

The example embeds these synthetic inputs.

```json
{
  "project": "046454",
  "asset": "LAB-17",
  "metric": "days since last report",
  "limit": 7,
  "comparison": "ge",
  "finding": "report due",
  "mode": "report",
  "primary": [
    8,
    6
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
    8,
    6
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
    8,
    6
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

Practice scheduled reporting using Terraform and check a repeatable reporting workflow in the school lab scenario.

## Conclusion

The project records an observable scheduled reporting result for training case `LAB-17`.

[Back to the Terraform project index](../README.md)

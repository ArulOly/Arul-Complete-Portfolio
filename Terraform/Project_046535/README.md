# Terraform: Access Review — Usability Improvement for Single-Host Lab (Project 046535)

## Project overview

One simulated lab host provides the setting for this access review exercise. This project includes source code, sample data, and instructions to run it. The readings are synthetic.

## Problem statement

One simulated lab host needs a repeatable check for unapproved grants.

## Project objectives

- Review unapproved grants and flag readings that meet the cutoff rule.
- Show the input source and flagged count in a concise result.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Terraform HCL, local values, input validation, and outputs.

## System architecture

`Synthetic observations → configuration → native evaluation → report`

## Features

- Review unapproved grants and flag readings that meet the cutoff rule.
- Show the input source and flagged count in a concise result.

## Prerequisites

Use Terraform 1.4 or later. Use a text editor and a local practice directory.

## Complete Terraform source code

```hcl
terraform {
  required_version = ">= 1.4"
}

variable "primary" {
  type    = list(number)
  default = [1]
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
  metric     = "unapproved grants"
  threshold  = 0
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
  description = "Plan-time findings for unapproved grants"
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

The comparison is gt with limit 0. A flagged row means access review needed. Show the input source and flagged count in a concise result.

## Sample input data

The example embeds these synthetic inputs.

```json
{
  "project": "046535",
  "asset": "HOST-03",
  "metric": "unapproved grants",
  "limit": 0,
  "comparison": "gt",
  "finding": "access review needed",
  "mode": "summary",
  "primary": [
    1
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

Practice access review using Terraform and check a repeatable usability improvement workflow in the single-host lab scenario.

## Conclusion

The project records an observable access review result for training case `HOST-03`.

[Back to the Terraform project index](../README.md)

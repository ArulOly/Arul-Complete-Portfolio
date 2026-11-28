# PowerShell: Automation — Reporting for Help Desk (Project 040894)

## Project overview

A help desk training queue provides the setting for this automation exercise. This project includes source code, sample data, and instructions to run it. The observations are synthetic.

## Problem statement

The help desk setting needs a reproducible way to check failed scheduled tasks.

## Project objectives

- Check failed scheduled tasks against the sample threshold.
- Produce a report for the supplied observations.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

PowerShell arrays, validation, comparison, and JSON.

## System architecture

`Synthetic fixture → native tool → reported result`

## Features

- Check failed scheduled tasks against the sample threshold.
- Produce a report for the supplied observations.

## Prerequisites

Use PowerShell 7. Use a text editor and a local practice directory.

## Complete PowerShell source code

```powershell
param([string[]]$Reading)
Set-StrictMode -Version Latest
$ErrorActionPreference = 'Stop'

$mode = 'report'
$rule = 'gt'
$threshold = 0
$primary = @(2,0)
$fallback = @()
if ($PSBoundParameters.ContainsKey('Reading')) { $primary = @($Reading) }
$source = 'primary'
$active = $primary
if ($active.Count -eq 0 -and $mode -eq 'recovery') {
    $source = 'fallback'
    $active = $fallback
}
if ($active.Count -eq 0) { throw 'Provide a reading.' }

$flagged = 0
foreach ($item in $active) {
    if ([string]$item -cnotmatch '^[0-9]{1,9}$') {
        throw 'Expected a nonnegative integer with at most nine digits.'
    }
    $number = [int]$item
    if (($rule -eq 'lt' -and $number -lt $threshold) -or
        ($rule -eq 'gt' -and $number -gt $threshold) -or
        ($rule -eq 'ge' -and $number -ge $threshold)) {
        $flagged++
    }
}
[ordered]@{source=$source; readings=$active.Count; flagged=$flagged} | ConvertTo-Json -Compress
```

## Implementation explanation

Check failed scheduled tasks against the sample threshold. Produce a report for the supplied observations.

## Sample input data

The code already includes these synthetic observations.

```json
{
  "project": "040894",
  "asset": "DESK-14",
  "metric": "failed scheduled tasks",
  "limit": 0,
  "comparison": "gt",
  "finding": "scheduled tasks need attention",
  "mode": "report",
  "primary": [
    2,
    0
  ],
  "fallback": []
}
```

## Execution instructions

Save the code as `practice.ps1` and run `pwsh -NoProfile -File practice.ps1`. Within PowerShell, use `./practice.ps1 -Reading 2,4` for a different primary sample.

## Example output

Illustrative excerpt for the synthetic sample.

```json
{
  "source": "primary",
  "readings": 2,
  "flagged": 1
}
```

## Testing and validation

Ran the script in PowerShell 7.5.4 and checked the JSON output. A malformed reading threw the expected validation error.

```json
{
  "source": "primary",
  "readings": 2,
  "flagged": 1
}
```

## Error handling

The program validates all input readings and reports a diagnostic for an invalid value.

## Troubleshooting

- Confirm the documented runtime and copy the complete source block.
- Start with the documented sample and use the filename or request path shown above.
- Compare the observed value with the example before changing an input.

## Limitations

The exercise uses portable PowerShell and does not call Windows-specific administrative commands.

## Learning outcomes

Practice automation using PowerShell and check a repeatable reporting workflow in the help desk scenario.

## Conclusion

The project records an observable automation result for training case `DESK-14`.

[Back to the PowerShell project index](../README.md)

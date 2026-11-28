# PowerShell: Performance Baseline — Reporting for Branch Office Simulator (Project 040574)

## Project overview

A simulated branch office provides the setting for this performance baseline exercise. This project includes source code, sample data, and instructions to run it. The observations are synthetic.

## Problem statement

The branch office simulator setting needs a reproducible way to check p95 latency in milliseconds.

## Project objectives

- Check p95 latency in milliseconds against the sample threshold.
- Produce a report for the supplied observations.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

PowerShell arrays, validation, comparison, and JSON.

## System architecture

`Synthetic fixture → native tool → reported result`

## Features

- Check p95 latency in milliseconds against the sample threshold.
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
$threshold = 80
$primary = @(96,80)
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

Check p95 latency in milliseconds against the sample threshold. Produce a report for the supplied observations.

## Sample input data

The code already includes these synthetic observations.

```json
{
  "project": "040574",
  "asset": "BRANCH-08",
  "metric": "p95 latency in milliseconds",
  "limit": 80,
  "comparison": "gt",
  "finding": "latency above baseline",
  "mode": "report",
  "primary": [
    96,
    80
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

Practice performance baseline using PowerShell and check a repeatable reporting workflow in the branch office simulator scenario.

## Conclusion

The project records an observable performance baseline result for training case `BRANCH-08`.

[Back to the PowerShell project index](../README.md)

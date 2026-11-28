# Bash: Recovery Test — Reporting for Public Dataset (Project 039969)

## Project overview

A synthetic public-data exercise provides the setting for this recovery test exercise. This project includes source code, sample data, and instructions to run it. The observations are synthetic.

## Problem statement

The public dataset setting needs a reproducible way to check unrestored records.

## Project objectives

- Check unrestored records against the sample threshold.
- Produce a report for the supplied observations.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Bash arrays, pattern matching, and arithmetic.

## System architecture

`Synthetic fixture → native tool → reported result`

## Features

- Check unrestored records against the sample threshold.
- Produce a report for the supplied observations.

## Prerequisites

Use Bash 5. Use a text editor and a local practice directory.

## Complete Bash source code

```bash
#!/usr/bin/env bash
set -euo pipefail
export LC_ALL=C

mode=report
threshold=0
operator=gt
primary=(1 0)
fallback=()
if (( $# )); then
  primary=("$@")
fi
source=primary
active=("${primary[@]}")
if (( ${#active[@]} == 0 )) && [[ $mode == recovery ]]; then
  active=("${fallback[@]}")
  source=fallback
fi
(( ${#active[@]} )) || { printf '%s\n' 'Provide at least one reading.' >&2; exit 2; }
flagged=0
for reading in "${active[@]}"; do
  if [[ ! $reading =~ ^[0-9]{1,9}$ ]]; then
    printf '%s\n' 'Expected a nonnegative integer with at most nine digits.' >&2
    exit 2
  fi
  number=$((10#$reading))
  case $operator in
    lt) if (( number < threshold )); then ((flagged+=1)); fi ;;
    gt) if (( number > threshold )); then ((flagged+=1)); fi ;;
    ge) if (( number >= threshold )); then ((flagged+=1)); fi ;;
  esac
done
printf '{"source":"%s","readings":%d,"flagged":%d}\n' "$source" "${#active[@]}" "$flagged"
```

## Implementation explanation

Check unrestored records against the sample threshold. Produce a report for the supplied observations.

## Sample input data

The code already includes these synthetic observations.

```json
{
  "project": "039969",
  "asset": "DATA-12",
  "metric": "unrestored records",
  "limit": 0,
  "comparison": "gt",
  "finding": "recovery remains incomplete",
  "mode": "report",
  "primary": [
    1,
    0
  ],
  "fallback": []
}
```

## Execution instructions

Save the code as `practice.sh` and run `bash practice.sh`. Pass integer arguments to try a different primary reading.

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

Ran the embedded script with Bash. The output matched the fixture and a nonnumeric reading returned status 2.

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

The local script evaluates integer observations supplied by the sample or command-line arguments.

## Learning outcomes

Practice recovery test using Bash and check a repeatable reporting workflow in the public dataset scenario.

## Conclusion

The project records an observable recovery test result for training case `DATA-12`.

[Back to the Bash project index](../README.md)

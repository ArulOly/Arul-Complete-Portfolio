# Bash: Visual Reporting — Recovery for School Lab (Project 039903)

## Project overview

Shared classroom equipment provides the setting for this visual reporting exercise. This project includes source code, sample data, and instructions to run it. The observations are synthetic.

## Problem statement

The school lab setting needs a reproducible way to check summary categories.

## Project objectives

- Check summary categories against the sample threshold.
- Use the sample fallback or inspect the recovered file.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Bash arrays, pattern matching, and arithmetic.

## System architecture

`Synthetic fixture → native tool → reported result`

## Features

- Check summary categories against the sample threshold.
- Use the sample fallback or inspect the recovered file.

## Prerequisites

Use Bash 5. Use a text editor and a local practice directory.

## Complete Bash source code

```bash
#!/usr/bin/env bash
set -euo pipefail
export LC_ALL=C

mode=recovery
threshold=5
operator=gt
primary=()
fallback=(6 5)
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

Check summary categories against the sample threshold. Use the sample fallback or inspect the recovered file.

## Sample input data

The code already includes these synthetic observations.

```json
{
  "project": "039903",
  "asset": "LAB-17",
  "metric": "summary categories",
  "limit": 5,
  "comparison": "gt",
  "finding": "chart needs a larger display",
  "mode": "recovery",
  "primary": [],
  "fallback": [
    6,
    5
  ]
}
```

## Execution instructions

Save the code as `practice.sh` and run `bash practice.sh`. Pass integer arguments to try a different primary reading.

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

Ran the embedded script with Bash. The output matched the fixture and a nonnumeric reading returned status 2.

```json
{
  "source": "fallback",
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

Practice visual reporting using Bash and check a repeatable recovery workflow in the school lab scenario.

## Conclusion

The project records an observable visual reporting result for training case `LAB-17`.

[Back to the Bash project index](../README.md)

# Raspberry Pi: Service Check — Automation for Single-Host Lab (Project 035632)

## Project overview

One simulated lab host provides the setting for this service check exercise. This project includes source code, sample data, and instructions to run it. The observations are synthetic.

## Problem statement

The single-host lab setting needs a reproducible way to check healthy service checks.

## Project objectives

- Check healthy service checks against the sample threshold.
- Process the supplied batch in one run.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Bash, Raspberry Pi OS device-tree model metadata, and synthetic readings.

## System architecture

`Synthetic fixture → native tool → reported result`

## Features

- Check healthy service checks against the sample threshold.
- Process the supplied batch in one run.

## Prerequisites

Use Raspberry Pi OS or a compatible Linux system with Bash. Use a text editor and a local practice directory.

## Complete Raspberry Pi source code

```bash
#!/usr/bin/env bash
set -euo pipefail
export LC_ALL=C
if [[ $(uname -s) != Linux ]]; then
  printf '%s
' 'Run this example on Raspberry Pi OS or compatible Linux.' >&2
  exit 2
fi
model_file=/proc/device-tree/model
if [[ -r $model_file ]]; then
  model=$(tr -d '\000' < "$model_file")
else
  model='Linux without device-tree model'
fi
printf 'Host model: %s
' "$model" >&2
mode=batch
threshold=3
operator=lt
primary=(2 4)
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

Check healthy service checks against the sample threshold. Process the supplied batch in one run.

## Sample input data

The code already includes these synthetic observations.

```json
{
  "project": "035632",
  "asset": "HOST-03",
  "metric": "healthy service checks",
  "limit": 3,
  "comparison": "lt",
  "finding": "health checks failed",
  "mode": "batch",
  "primary": [
    2,
    4
  ],
  "fallback": []
}
```

## Execution instructions

Save the code as `practice.sh` and run `bash practice.sh`. The model line appears on standard error and the result on standard output.

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

Ran the Bash script on Linux and checked its portable model fallback, sample output, and malformed input. A synthetic device-tree file verified model parsing. Physical Raspberry Pi hardware remains unverified.

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

The readings are synthetic. This exercise reads the model when the system exposes it and does not access GPIO.

## Learning outcomes

Practice service check using Raspberry Pi and check a repeatable automation workflow in the single-host lab scenario.

## Conclusion

The project records an observable service check result for training case `HOST-03`.

[Back to the Raspberry Pi project index](../README.md)

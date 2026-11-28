# Docker: Access Review — Automation for School Lab (Project 017502)

## Project overview

Shared classroom equipment provides the setting for this access review exercise. This project includes source code, sample data, and instructions to run it. The readings are synthetic.

## Problem statement

Shared classroom equipment needs a repeatable check for unapproved grants.

## Project objectives

- Review unapproved grants and flag readings that meet the cutoff rule.
- Evaluate the readings in input order.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Docker CLI and POSIX shell with BusyBox awk.

## System architecture

`Synthetic observations → container command → native evaluation → report`

## Features

- Review unapproved grants and flag readings that meet the cutoff rule.
- Evaluate the readings in input order.

## Prerequisites

Use Docker and the official Alpine 3.22 image. Use a text editor and a local practice directory.

## Complete Docker source code

```sh
docker run --rm -i --network none --read-only --user 10001:10001 alpine:3.22 sh -s <<'PORTFOLIO'
#!/bin/sh
set -eu
primary=${PRIMARY-'1 0'}
fallback=${FALLBACK-''}
threshold=${THRESHOLD-0}
comparison=gt
mode=batch
metric='unapproved grants'

source=primary
values=$primary
if [ -z "$values" ] && [ "$mode" = recovery ]; then
    source=fallback
    values=$fallback
fi

printf '%s\n' "$values" | awk -v limit="$threshold" -v op="$comparison" \
    -v source="$source" -v mode="$mode" -v metric="$metric" '
    BEGIN { if (limit !~ /^[0-9]+$/ || limit > 1000000000) exit 2 }
    {
        if (NF == 0) exit 2
        for (i = 1; i <= NF; i++) {
            if ($i !~ /^[0-9]+$/ || $i > 1000000000) exit 2
            n = $i + 0
            flagged += (op == "lt" && n < limit) || (op == "gt" && n > limit) || (op == "ge" && n >= limit)
            detail = detail (i == 1 ? "" : ",") n
        }
        printf "{\"source\":\"%s\",\"readings\":%d,\"flagged\":%d", source, NF, flagged
        if (mode == "baseline" || mode == "report") printf ",\"values\":[%s]", detail
        printf "}\n"
    }
' || { printf '%s\n' "Invalid $metric input" >&2; exit 2; }
PORTFOLIO
```

## Implementation explanation

The comparison is gt with limit 0. A flagged row means access review needed. Evaluate the readings in input order.

## Sample input data

The example embeds these synthetic inputs.

```json
{
  "project": "017502",
  "asset": "LAB-17",
  "metric": "unapproved grants",
  "limit": 0,
  "comparison": "gt",
  "finding": "access review needed",
  "mode": "batch",
  "primary": [
    1,
    0
  ],
  "fallback": []
}
```

## Execution instructions

Save the block as `run.sh` and run `sh run.sh`. Docker runs the embedded check inside Alpine without network access after the image is available.

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

Executed the embedded check inside the Alpine 3.22 Docker image. The expected result matched the container output, and invalid input returned status 2.

```json
{
  "source": "primary",
  "readings": 2,
  "flagged": 1
}
```

## Error handling

The implementation rejects invalid numeric inputs. Recovery uses the fallback only when the primary list is empty.

## Troubleshooting

- Confirm the documented runtime and copy the complete source block.
- Start with the documented sample and use the filename or request path shown above.
- Compare the observed value with the example before changing an input.

## Limitations

The readings are synthetic and the container exits after printing its result.

## Learning outcomes

Practice access review using Docker and check a repeatable automation workflow in the school lab scenario.

## Conclusion

The project records an observable access review result for training case `LAB-17`.

[Back to the Docker project index](../README.md)

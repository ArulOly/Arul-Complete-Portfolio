# Docker: Recovery Test — Recovery for Portfolio Demo (Project 017998)

## Project overview

A repeatable portfolio demonstration provides the setting for this recovery test exercise. This project includes source code, sample data, and instructions to run it. The readings are synthetic.

## Problem statement

A repeatable portfolio demonstration needs a repeatable check for unrestored records.

## Project objectives

- Review unrestored records and flag readings that meet the cutoff rule.
- Use the fallback observations when the primary set is empty.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Docker CLI and POSIX shell with BusyBox awk.

## System architecture

`Synthetic observations → container command → native evaluation → report`

## Features

- Review unrestored records and flag readings that meet the cutoff rule.
- Use the fallback observations when the primary set is empty.

## Prerequisites

Use Docker and the official Alpine 3.22 image. Use a text editor and a local practice directory.

## Complete Docker source code

```sh
docker run --rm -i --network none --read-only --user 10001:10001 alpine:3.22 sh -s <<'PORTFOLIO'
#!/bin/sh
set -eu
primary=${PRIMARY-''}
fallback=${FALLBACK-'1 0'}
threshold=${THRESHOLD-0}
comparison=gt
mode=recovery
metric='unrestored records'

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

The comparison is gt with limit 0. A flagged row means recovery remains incomplete. Use the fallback observations when the primary set is empty.

## Sample input data

The example embeds these synthetic inputs.

```json
{
  "project": "017998",
  "asset": "DEMO-02",
  "metric": "unrestored records",
  "limit": 0,
  "comparison": "gt",
  "finding": "recovery remains incomplete",
  "mode": "recovery",
  "primary": [],
  "fallback": [
    1,
    0
  ]
}
```

## Execution instructions

Save the block as `run.sh` and run `sh run.sh`. Docker runs the embedded check inside Alpine without network access after the image is available.

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

Executed the embedded check inside the Alpine 3.22 Docker image. The expected result matched the container output, and invalid input returned status 2.

```json
{
  "source": "fallback",
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

Practice recovery test using Docker and check a repeatable recovery workflow in the portfolio demo scenario.

## Conclusion

The project records an observable recovery test result for training case `DEMO-02`.

[Back to the Docker project index](../README.md)

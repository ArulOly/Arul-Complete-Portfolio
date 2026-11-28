# Git: Monitoring — Reporting for Small Business (Project 038059)

## Project overview

Local shop operations provides the setting for this monitoring exercise. This project includes source code, sample data, and instructions to run it. The observations are synthetic.

## Problem statement

The small business setting needs a reproducible way to check peak utilization percent.

## Project objectives

- Track and inspect the synthetic observation file with local Git operations.
- Inspect the diff between the two sample commits.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Local Git commits, history, diffs, restore, and status.

## System architecture

`Synthetic fixture → native tool → reported result`

## Features

- Track and inspect the synthetic observation file with local Git operations.
- Inspect the diff between the two sample commits.

## Prerequisites

Use Git 2.28 or later and a POSIX shell. Use a text editor and a local practice directory.

## Complete Git source code

```sh
#!/bin/sh
set -eu
export LC_ALL=C
repo=$(mktemp -d)
trap 'rm -rf "$repo"' EXIT
export GIT_CONFIG_NOSYSTEM=1
export GIT_CONFIG_GLOBAL=/dev/null
git init -q --template= --initial-branch=main "$repo"
git -C "$repo" config user.name 'Portfolio Practice'
git -C "$repo" config user.email 'practice@example.invalid'
git -C "$repo" config core.hooksPath /dev/null

printf '%s\n' '83 80' > "$repo/observations.txt"
git -C "$repo" add -- observations.txt
git -C "$repo" commit -qm baseline
printf '%s\n' '83 80 80' > "$repo/observations.txt"
git -C "$repo" add -- observations.txt
git -C "$repo" commit -qm update
git -C "$repo" diff HEAD^ HEAD -- observations.txt
```

## Implementation explanation

Track and inspect the synthetic observation file with local Git operations. Inspect the diff between the two sample commits.

## Sample input data

The code already includes these synthetic observations.

```json
{
  "project": "038059",
  "asset": "SHOP-22",
  "metric": "peak utilization percent",
  "limit": 80,
  "comparison": "gt",
  "finding": "utilization above budget",
  "mode": "report",
  "primary": [
    83,
    80
  ],
  "fallback": []
}
```

## Execution instructions

Save the code as `practice.sh` and run `sh practice.sh`. The script creates and cleans up an isolated temporary repository.

## Example output

Illustrative excerpt for the synthetic sample.

```json
{
  "removed": "83 80",
  "added": "83 80 80"
}
```

## Testing and validation

Ran the embedded script twice with Git. The history, file contents, diff, or clean status matched the expected result.

```json
{
  "removed": "83 80",
  "added": "83 80 80"
}
```

## Error handling

Git reports a failed command and the temporary repository is cleaned up.

## Troubleshooting

- Confirm the documented runtime and copy the complete source block.
- Start with the documented sample and use the filename or request path shown above.
- Compare the observed value with the example before changing an input.

## Limitations

The temporary repository contains only fictional readings and has no shared remote.

## Learning outcomes

Practice monitoring using Git and check a repeatable reporting workflow in the small business scenario.

## Conclusion

The project records an observable monitoring result for training case `SHOP-22`.

[Back to the Git project index](../README.md)

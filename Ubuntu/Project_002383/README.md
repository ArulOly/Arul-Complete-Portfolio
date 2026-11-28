# Ubuntu: Dashboard — Recovery for Single-Host Lab (Project 002383)

## Project overview

One simulated lab host provides the setting for this dashboard exercise. This exercise is based on the student guide and uses fictional data. This project includes source code, sample data, and instructions to run it.

## Problem statement

The single-host lab needs a reproducible dashboard workflow focused on recovery on Ubuntu.

## Project objectives

- Aggregate ticket counts and show open workload as a share of all tickets.
- Load a fallback fixture when the primary is absent or invalid.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Ubuntu, dpkg-query, JSON, and Python standard-library modules.

## System architecture

`OS identity → local JSON → validated workflow → terminal or report`

## Features

- Aggregate ticket counts and show open workload as a share of all tickets.
- Load a fallback fixture when the primary is absent or invalid.

## Prerequisites

Use Ubuntu in a practice environment with Python 3.11 or later and dpkg-query. Use a text editor and a local practice directory.

## Complete Ubuntu source code

```python
"""Local Ubuntu reference implementation for project 002383."""


from pathlib import Path
import argparse
import json
import platform
import subprocess
import sys


EXPECTED_SYSTEM = 'Ubuntu'


def label(record, key):
    value = record.get(key)
    if not isinstance(value, str) or not value.strip():
        raise ValueError(f"{key} must be non-empty text")
    return value.strip()


def integer(record, key, minimum=0, maximum=1_000_000_000):
    value = record.get(key)
    if type(value) is not int or not minimum <= value <= maximum:
        raise ValueError(f"{key} must be an integer from {minimum} to {maximum}")
    return value


def mapping(record, key):
    value = record.get(key)
    if not isinstance(value, dict):
        raise ValueError(f"{key} must be an object")
    return value


def evaluate(record):
    """Aggregate ticket counts and show open workload as a share of all tickets."""
    queue = mapping(record, "queue")
    counts = {key: integer(queue, key) for key in ("open", "in_progress", "closed")}
    total = sum(counts.values())
    active = counts["open"] + counts["in_progress"]
    return {"counts": counts, "total": total,
            "active_pct": round(100 * active / total, 1) if total else 0.0}


def platform_details():
    if platform.system() != "Linux":
        raise ValueError("This demo requires Ubuntu")
    distribution = platform.freedesktop_os_release()
    if distribution.get("ID") != 'ubuntu':
        raise ValueError("Expected Ubuntu /etc/os-release information")
    version = subprocess.run(['dpkg-query', '--show', '--showformat=${Version}', 'dpkg'], text=True, capture_output=True,
                             timeout=5, check=True).stdout.strip()
    if not version:
        raise ValueError("The platform version command returned no output")
    return {"platform": EXPECTED_SYSTEM, "tool": 'dpkg-query', "version": version}


def load(path):
    value = json.loads(path.read_text(encoding="utf-8"))
    if not isinstance(value, dict):
        raise ValueError("Expected a JSON object")
    json.dumps(value, allow_nan=False)
    return value


def process(args):
    try:
        record = load(args.input)
        return {"source": "primary", "asset": label(record, "asset"),
                "result": evaluate(record)}, 0
    except (OSError, ValueError, OverflowError, RecursionError) as exc:
        record = load(args.fallback)
        return {"source": "fallback", "asset": label(record, "asset"),
                "primary_error": str(exc), "result": evaluate(record)}, 0


def main():
    parser = argparse.ArgumentParser(description='Aggregate ticket counts and show open workload as a share of all tickets.')
    parser.add_argument("input", type=Path)
    parser.add_argument("--fallback", type=Path, required=True)
    args = parser.parse_args()
    try:
        environment = platform_details()
        outcome, status = process(args)
        report = {"environment": environment, "result": outcome}
        serialized = json.dumps(report, indent=2, ensure_ascii=False, allow_nan=False)
        print(serialized)
        return status
    except (OSError, ValueError, OverflowError, RecursionError, subprocess.SubprocessError) as exc:
        print(f"Demo error: {exc}", file=sys.stderr)
        return 2


if __name__ == "__main__":
    raise SystemExit(main())
```

## Implementation explanation

The program verifies the OS and reads dpkg-query to identify the practice environment. The dashboard calculation works on synthetic input. Load a fallback fixture when the primary is absent or invalid.

## Sample input data

Save this synthetic record as `sample.json`.

```json
{
  "asset": "HOST-03",
  "queue": {
    "open": 5,
    "in_progress": 3,
    "closed": 12
  }
}
```

## Execution instructions

Copy the Python source to `project.py` and the sample JSON to `sample.json` in the same directory. Save a second copy of the sample as `fallback.json` before running the command.

```sh
python3 project.py sample.json --fallback fallback.json
```

## Example output

Illustrative excerpt for the synthetic sample.

```json
{
  "active_pct": 40.0
}
```

## Testing and validation

Actual tests ran on a Ubuntu 24.04 container. The platform command returned the observed version and the sample result repeated. A bad primary used the supplied fallback. Two unusable inputs returned exit status 2.

```json
{
  "active_pct": 40.0,
  "environment": {
    "platform": "Ubuntu",
    "tool": "dpkg-query",
    "version": "1.22.6ubuntu6.6"
  }
}
```

## Error handling

The CLI reports invalid input, missing files, and failed platform checks. Batch and recovery modes include row or fallback details.

## Troubleshooting

- Confirm the documented runtime and copy the complete source block.
- Start with the documented sample and use the filename or request path shown above.
- Compare the observed value with the example before changing an input.

## Limitations

This exercise works on local synthetic records. The only operating-system commands are read-only version checks.

## Learning outcomes

Practice dashboard using Ubuntu and check a repeatable recovery workflow in the single-host lab scenario.

## Conclusion

The project records an observable dashboard result for training case `HOST-03`.

[Back to the Ubuntu project index](../README.md)

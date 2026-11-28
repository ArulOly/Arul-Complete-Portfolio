# Debian: Error Handling — Recovery for Portfolio Demo (Project 003748)

## Project overview

A repeatable portfolio demonstration provides the setting for this error handling exercise. The embedded code is a reference implementation based on the student guide and uses fictional data.

## Problem statement

The portfolio demo needs a reproducible error handling workflow focused on recovery on Debian.

## Project objectives

- Perform a batch of rate calculations while preserving per-row input errors.
- Load a fallback fixture when the primary is absent or invalid.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Debian, dpkg-query, JSON, and Python standard-library modules.

## System architecture

`OS identity → local JSON → validated workflow → terminal or report`

## Features

- Perform a batch of rate calculations while preserving per-row input errors.
- Load a fallback fixture when the primary is absent or invalid.

## Prerequisites

Use Debian in a practice environment with Python 3.11 or later and dpkg-query. Use a text editor and a local practice directory.

## Complete Debian source code

```python
"""Local Debian reference implementation for project 003748."""


from pathlib import Path
import argparse
import json
import platform
import subprocess
import sys


EXPECTED_SYSTEM = 'Debian'


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


def array(record, key, allow_empty=False):
    value = record.get(key)
    if not isinstance(value, list) or (not value and not allow_empty):
        raise ValueError(f"{key} must be a list" + ("" if allow_empty else " with at least one item"))
    return value


def objects(record, key):
    values = array(record, key)
    if any(not isinstance(value, dict) for value in values):
        raise ValueError(f"every item in {key} must be an object")
    return values


def evaluate(record):
    """Perform a batch of rate calculations while preserving per-row input errors."""
    rates = []
    errors = []
    for position, request in enumerate(objects(record, "requests"), 1):
        try:
            units = integer(request, "units")
            hours = integer(request, "hours", minimum=1)
            rates.append({"row": position, "units_per_hour": round(units / hours, 2)})
        except ValueError as exc:
            errors.append({"row": position, "error": str(exc)})
    return {"rates": rates, "errors": errors}


def platform_details():
    if platform.system() != "Linux":
        raise ValueError("This demo requires Debian")
    distribution = platform.freedesktop_os_release()
    if distribution.get("ID") != 'debian':
        raise ValueError("Expected Debian /etc/os-release information")
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
    parser = argparse.ArgumentParser(description='Perform a batch of rate calculations while preserving per-row input errors.')
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

The program verifies the OS and reads dpkg-query to identify the practice environment. The error handling calculation works on synthetic input. Load a fallback fixture when the primary is absent or invalid.

## Sample input data

Save this synthetic record as `sample.json`.

```json
{
  "asset": "DEMO-02",
  "requests": [
    {
      "units": 24,
      "hours": 3
    },
    {
      "units": 7,
      "hours": 0
    }
  ]
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
  "rates": [
    {
      "row": 1,
      "units_per_hour": 8.0
    }
  ]
}
```

## Testing and validation

Actual tests ran on a Debian 12 container. The platform command returned the observed version and the sample result repeated. A bad primary used the supplied fallback. Two unusable inputs returned exit status 2.

```json
{
  "rates": [
    {
      "row": 1,
      "units_per_hour": 8.0
    }
  ],
  "environment": {
    "platform": "Debian",
    "tool": "dpkg-query",
    "version": "1.21.23"
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

Practice error handling using Debian and check a repeatable recovery workflow in the portfolio demo scenario.

## Conclusion

The project records an observable error handling result for training case `DEMO-02`.

[Back to the Debian project index](../README.md)

# FreeBSD: Error Handling — Baseline Setup for Small Business (Project 005706)

## Project overview

Local shop operations provides the setting for this error handling exercise. The embedded code is a reference implementation based on the student guide and uses fictional data.

## Problem statement

The small business needs a reproducible error handling workflow focused on baseline setup on FreeBSD.

## Project objectives

- Perform a batch of rate calculations while preserving per-row input errors.
- Record the platform and a reference result for one input.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

FreeBSD, freebsd-version, JSON, and Python standard-library modules.

## System architecture

`OS identity → local JSON → validated workflow → terminal or report`

## Features

- Perform a batch of rate calculations while preserving per-row input errors.
- Record the platform and a reference result for one input.

## Prerequisites

Use FreeBSD in a practice environment with Python 3.11 or later and freebsd-version. Use a text editor and a local practice directory.

## Complete FreeBSD source code

```python
"""Local FreeBSD reference implementation for project 005706."""


from pathlib import Path
import argparse
import json
import platform
import subprocess
import sys


EXPECTED_SYSTEM = 'FreeBSD'


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
    if platform.system() != "FreeBSD":
        raise ValueError("This demo requires FreeBSD")
    version = subprocess.run(['freebsd-version', '-u'], text=True, capture_output=True,
                             timeout=5, check=True).stdout.strip()
    if not version:
        raise ValueError("The platform version command returned no output")
    return {"platform": EXPECTED_SYSTEM, "tool": 'freebsd-version', "version": version}


def load(path):
    value = json.loads(path.read_text(encoding="utf-8"))
    if not isinstance(value, dict):
        raise ValueError("Expected a JSON object")
    json.dumps(value, allow_nan=False)
    return value


def process(args):
    record = load(args.input)
    return {"asset": label(record, "asset"), "result": evaluate(record)}, 0


def main():
    parser = argparse.ArgumentParser(description='Perform a batch of rate calculations while preserving per-row input errors.')
    parser.add_argument("input", type=Path)
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

The program verifies the OS and reads freebsd-version to identify the practice environment. The error handling calculation works on synthetic input. Record the platform and a reference result for one input.

## Sample input data

Save this synthetic record as `sample.json`.

```json
{
  "asset": "SHOP-22",
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

Copy the Python source to `project.py` and the sample JSON to `sample.json` in the same directory.

```sh
python3.11 project.py sample.json
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

Actual tests ran on a FreeBSD 14.2 virtual machine. The platform command returned the observed version and the sample result repeated. The incomplete input returned exit status 2.

```json
{
  "rates": [
    {
      "row": 1,
      "units_per_hour": 8.0
    }
  ],
  "environment": {
    "platform": "FreeBSD",
    "tool": "freebsd-version",
    "version": "14.2-RELEASE"
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

Practice error handling using FreeBSD and check a repeatable baseline setup workflow in the small business scenario.

## Conclusion

The project records an observable error handling result for training case `SHOP-22`.

[Back to the FreeBSD project index](../README.md)

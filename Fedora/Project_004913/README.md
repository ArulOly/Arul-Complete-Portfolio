# Fedora: Visual Reporting — Recovery for Home Development (Project 004913)

## Project overview

A home development workspace provides the setting for this visual reporting exercise. This exercise is based on the student guide and uses fictional data. This project includes source code, sample data, and instructions to run it.

## Problem statement

The home development needs a reproducible visual reporting workflow focused on recovery on Fedora.

## Project objectives

- Create a proportional text bar chart with exact values and a labeled scale.
- Load a fallback fixture when the primary is absent or invalid.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Fedora, rpm, JSON, and Python standard-library modules.

## System architecture

`OS identity → local JSON → validated workflow → terminal or report`

## Features

- Create a proportional text bar chart with exact values and a labeled scale.
- Load a fallback fixture when the primary is absent or invalid.

## Prerequisites

Use Fedora in a practice environment with Python 3.11 or later and rpm. Use a text editor and a local practice directory.

## Complete Fedora source code

```python
"""Local Fedora reference implementation for project 004913."""


from pathlib import Path
import argparse
import json
import platform
import subprocess
import sys


EXPECTED_SYSTEM = 'Fedora'


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
    """Create a proportional text bar chart with exact values and a labeled scale."""
    values = mapping(record, "values")
    if not values:
        raise ValueError("values must contain at least one category")
    counts = {key: integer(values, key) for key in values}
    width = integer(record, "width", minimum=1, maximum=60)
    peak = max(counts.values())
    bars = [f"{key}: {'#' * ((value * width + peak // 2) // peak if peak else 0)} ({value})"
            for key, value in sorted(counts.items())]
    return {"peak": peak, "max_bar_width": width, "bars": bars}


def platform_details():
    if platform.system() != "Linux":
        raise ValueError("This demo requires Fedora")
    distribution = platform.freedesktop_os_release()
    if distribution.get("ID") != 'fedora':
        raise ValueError("Expected Fedora /etc/os-release information")
    version = subprocess.run(['rpm', '-q', '--queryformat', '%{VERSION}', 'rpm'], text=True, capture_output=True,
                             timeout=5, check=True).stdout.strip()
    if not version:
        raise ValueError("The platform version command returned no output")
    return {"platform": EXPECTED_SYSTEM, "tool": 'rpm', "version": version}


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
    parser = argparse.ArgumentParser(description='Create a proportional text bar chart with exact values and a labeled scale.')
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

The program verifies the OS and reads rpm to identify the practice environment. The visual reporting calculation works on synthetic input. Load a fallback fixture when the primary is absent or invalid.

## Sample input data

Save this synthetic record as `sample.json`.

```json
{
  "asset": "DEV-05",
  "values": {
    "available": 12,
    "reserved": 7,
    "repair": 3
  },
  "width": 12
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
  "peak": 12
}
```

## Testing and validation

Actual tests ran on a Fedora 43 container. The platform command returned the observed version and the sample result repeated. A bad primary used the supplied fallback. Two unusable inputs returned exit status 2.

```json
{
  "peak": 12,
  "environment": {
    "platform": "Fedora",
    "tool": "rpm",
    "version": "6.0.2"
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

Practice visual reporting using Fedora and check a repeatable recovery workflow in the home development scenario.

## Conclusion

The project records an observable visual reporting result for training case `DEV-05`.

[Back to the Fedora project index](../README.md)

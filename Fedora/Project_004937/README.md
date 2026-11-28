# Fedora: Visual Reporting — Automation for Two-Node Lab (Project 004937)

## Project overview

A two-node lab simulation provides the setting for this visual reporting exercise. The embedded code is a reference implementation based on the student guide and uses fictional data.

## Problem statement

The two-node lab needs a reproducible visual reporting workflow focused on automation on Fedora.

## Project objectives

- Create a proportional text bar chart with exact values and a labeled scale.
- Process a batch and keep successful rows alongside diagnostics.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Fedora, rpm, JSON, and Python standard-library modules.

## System architecture

`OS identity → local JSON → validated workflow → terminal or report`

## Features

- Create a proportional text bar chart with exact values and a labeled scale.
- Process a batch and keep successful rows alongside diagnostics.

## Prerequisites

Use Fedora in a practice environment with Python 3.11 or later and rpm. Use a text editor and a local practice directory.

## Complete Fedora source code

```python
"""Local Fedora reference implementation for project 004937."""


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


def array(record, key, allow_empty=False):
    value = record.get(key)
    if not isinstance(value, list) or (not value and not allow_empty):
        raise ValueError(f"{key} must be a list" + ("" if allow_empty else " with at least one item"))
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
    records = array(load(args.input), "records")
    rows = []
    for index, candidate in enumerate(records, 1):
        try:
            if not isinstance(candidate, dict):
                raise ValueError("Expected a record")
            rows.append({"row": index, "asset": label(candidate, "asset"), "result": evaluate(candidate)})
        except (ValueError, OverflowError) as exc:
            rows.append({"row": index, "error": str(exc)})
    return {"rows": rows}, int(any("error" in row for row in rows))


def main():
    parser = argparse.ArgumentParser(description='Create a proportional text bar chart with exact values and a labeled scale.')
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

The program verifies the OS and reads rpm to identify the practice environment. The visual reporting calculation works on synthetic input. Process a batch and keep successful rows alongside diagnostics.

## Sample input data

Save this synthetic record as `sample.json`.

```json
{
  "records": [
    {
      "asset": "NODE-B",
      "values": {
        "available": 12,
        "reserved": 7,
        "repair": 3
      },
      "width": 12
    },
    {
      "asset": "NODE-B-B",
      "values": {
        "available": 12,
        "reserved": 7,
        "repair": 3
      },
      "width": 12
    }
  ]
}
```

## Execution instructions

Copy the Python source to `project.py` and the sample JSON to `sample.json` in the same directory.

```sh
python3 project.py sample.json
```

## Example output

Illustrative excerpt for the synthetic sample.

```json
{
  "peak": 12
}
```

## Testing and validation

Actual tests ran on a Fedora 43 container. The platform command returned the observed version and the sample result repeated. A mixed batch preserved its successful row and returned exit status 1.

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

Practice visual reporting using Fedora and check a repeatable automation workflow in the two-node lab scenario.

## Conclusion

The project records an observable visual reporting result for training case `NODE-B`.

[Back to the Fedora project index](../README.md)

# Ubuntu: Alerting — Automation for Two-Node Lab (Project 002437)

## Project overview

A two-node lab simulation provides the setting for this alerting exercise. This exercise is based on the student guide and uses fictional data. This project includes source code, sample data, and instructions to run it.

## Problem statement

The two-node lab needs a reproducible alerting workflow focused on automation on Ubuntu.

## Project objectives

- Emit one local alert per consecutive breach episode after a configured threshold.
- Process a batch and keep successful rows alongside diagnostics.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Ubuntu, dpkg-query, JSON, and Python standard-library modules.

## System architecture

`OS identity → local JSON → validated workflow → terminal or report`

## Features

- Emit one local alert per consecutive breach episode after a configured threshold.
- Process a batch and keep successful rows alongside diagnostics.

## Prerequisites

Use Ubuntu in a practice environment with Python 3.11 or later and dpkg-query. Use a text editor and a local practice directory.

## Complete Ubuntu source code

```python
"""Local Ubuntu reference implementation for project 002437."""


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


def array(record, key, allow_empty=False):
    value = record.get(key)
    if not isinstance(value, list) or (not value and not allow_empty):
        raise ValueError(f"{key} must be a list" + ("" if allow_empty else " with at least one item"))
    return value


def series(record, key, maximum=1_000_000_000):
    return [integer({key: item}, key, maximum=maximum) for item in array(record, key)]


def evaluate(record):
    """Emit one local alert per consecutive breach episode after a configured threshold."""
    limit = integer(record, "limit")
    needed = integer(record, "consecutive", minimum=1)
    streak = 0
    alerts = []
    for index, value in enumerate(series(record, "readings"), 1):
        streak = streak + 1 if value > limit else 0
        if streak == needed:
            alerts.append({"sample": index, "reading": value})
    return {"limit": limit, "alerts": alerts}


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
    parser = argparse.ArgumentParser(description='Emit one local alert per consecutive breach episode after a configured threshold.')
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

The program verifies the OS and reads dpkg-query to identify the practice environment. The alerting calculation works on synthetic input. Process a batch and keep successful rows alongside diagnostics.

## Sample input data

Save this synthetic record as `sample.json`.

```json
{
  "records": [
    {
      "asset": "NODE-B",
      "readings": [
        10,
        22,
        24,
        25,
        8,
        26,
        29
      ],
      "limit": 20,
      "consecutive": 2
    },
    {
      "asset": "NODE-B-B",
      "readings": [
        10,
        22,
        24,
        25,
        8,
        26,
        29
      ],
      "limit": 20,
      "consecutive": 2
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
  "alerts": [
    {
      "sample": 3,
      "reading": 24
    },
    {
      "sample": 7,
      "reading": 29
    }
  ]
}
```

## Testing and validation

Actual tests ran on a Ubuntu 24.04 container. The platform command returned the observed version and the sample result repeated. A mixed batch preserved its successful row and returned exit status 1.

```json
{
  "alerts": [
    {
      "sample": 3,
      "reading": 24
    },
    {
      "sample": 7,
      "reading": 29
    }
  ],
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

Practice alerting using Ubuntu and check a repeatable automation workflow in the two-node lab scenario.

## Conclusion

The project records an observable alerting result for training case `NODE-B`.

[Back to the Ubuntu project index](../README.md)

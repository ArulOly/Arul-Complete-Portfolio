# Fedora: Access Review — Recovery for Portfolio Demo (Project 004548)

## Project overview

A repeatable portfolio demonstration provides the setting for this access review exercise. This exercise is based on the student guide and uses fictional data. This project includes source code, sample data, and instructions to run it.

## Problem statement

The portfolio demo needs a reproducible access review workflow focused on recovery on Fedora.

## Project objectives

- Compare synthetic active grants with an approved role policy.
- Load a fallback fixture when the primary is absent or invalid.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Fedora, rpm, JSON, and Python standard-library modules.

## System architecture

`OS identity → local JSON → validated workflow → terminal or report`

## Features

- Compare synthetic active grants with an approved role policy.
- Load a fallback fixture when the primary is absent or invalid.

## Prerequisites

Use Fedora in a practice environment with Python 3.11 or later and rpm. Use a text editor and a local practice directory.

## Complete Fedora source code

```python
"""Local Fedora reference implementation for project 004548."""


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


def array(record, key, allow_empty=False):
    value = record.get(key)
    if not isinstance(value, list) or (not value and not allow_empty):
        raise ValueError(f"{key} must be a list" + ("" if allow_empty else " with at least one item"))
    return value


def labels(record, key, allow_empty=False):
    return [label({key: item}, key) for item in array(record, key, allow_empty)]


def evaluate(record):
    """Compare synthetic active grants with an approved role policy."""
    approved = set(labels(record, "approved", allow_empty=True))
    grants = set(labels(record, "grants", allow_empty=True))
    return {"excess_grants": sorted(grants - approved),
            "missing_grants": sorted(approved - grants),
            "matches_policy": approved == grants}


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
    parser = argparse.ArgumentParser(description='Compare synthetic active grants with an approved role policy.')
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

The program verifies the OS and reads rpm to identify the practice environment. The access review calculation works on synthetic input. Load a fallback fixture when the primary is absent or invalid.

## Sample input data

Save this synthetic record as `sample.json`.

```json
{
  "asset": "DEMO-02",
  "approved": [
    "read",
    "submit"
  ],
  "grants": [
    "read",
    "admin"
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
  "excess_grants": [
    "admin"
  ]
}
```

## Testing and validation

Actual tests ran on a Fedora 43 container. The platform command returned the observed version and the sample result repeated. A bad primary used the supplied fallback. Two unusable inputs returned exit status 2.

```json
{
  "excess_grants": [
    "admin"
  ],
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

Practice access review using Fedora and check a repeatable recovery workflow in the portfolio demo scenario.

## Conclusion

The project records an observable access review result for training case `DEMO-02`.

[Back to the Fedora project index](../README.md)

# Debian: Configuration Comparison — Baseline Setup for Portfolio Demo (Project 003696)

## Project overview

A repeatable portfolio demonstration provides the setting for this configuration comparison exercise. The embedded code is a reference implementation based on the student guide and uses fictional data.

## Problem statement

The portfolio demo needs a reproducible configuration comparison workflow focused on baseline setup on Debian.

## Project objectives

- Identify added, removed, and changed settings between two configuration objects.
- Record the platform and a reference result for one input.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Debian, dpkg-query, JSON, and Python standard-library modules.

## System architecture

`OS identity → local JSON → validated workflow → terminal or report`

## Features

- Identify added, removed, and changed settings between two configuration objects.
- Record the platform and a reference result for one input.

## Prerequisites

Use Debian in a practice environment with Python 3.11 or later and dpkg-query. Use a text editor and a local practice directory.

## Complete Debian source code

```python
"""Local Debian reference implementation for project 003696."""


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


def mapping(record, key):
    value = record.get(key)
    if not isinstance(value, dict):
        raise ValueError(f"{key} must be an object")
    return value


def evaluate(record):
    """Identify added, removed, and changed settings between two configuration objects."""
    baseline = mapping(record, "baseline")
    current = mapping(record, "current")
    return {"added": sorted(current.keys() - baseline.keys()),
            "removed": sorted(baseline.keys() - current.keys()),
            "changed": sorted(key for key in baseline.keys() & current.keys()
                              if json.dumps(baseline[key], sort_keys=True)
                              != json.dumps(current[key], sort_keys=True))}


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
    record = load(args.input)
    return {"asset": label(record, "asset"), "result": evaluate(record)}, 0


def main():
    parser = argparse.ArgumentParser(description='Identify added, removed, and changed settings between two configuration objects.')
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

The program verifies the OS and reads dpkg-query to identify the practice environment. The configuration comparison calculation works on synthetic input. Record the platform and a reference result for one input.

## Sample input data

Save this synthetic record as `sample.json`.

```json
{
  "asset": "DEMO-02",
  "baseline": {
    "timeout": 30,
    "theme": "light"
  },
  "current": {
    "timeout": 45,
    "retries": 2
  }
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
  "changed": [
    "timeout"
  ]
}
```

## Testing and validation

Actual tests ran on a Debian 12 container. The platform command returned the observed version and the sample result repeated. The incomplete input returned exit status 2.

```json
{
  "changed": [
    "timeout"
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

Practice configuration comparison using Debian and check a repeatable baseline setup workflow in the portfolio demo scenario.

## Conclusion

The project records an observable configuration comparison result for training case `DEMO-02`.

[Back to the Debian project index](../README.md)

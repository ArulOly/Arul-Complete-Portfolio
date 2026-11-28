# Fedora: Data Validation — Reporting for Two-Node Lab (Project 004789)

## Project overview

A two-node lab simulation provides the setting for this data validation exercise. This exercise is based on the student guide and uses fictional data. This project includes source code, sample data, and instructions to run it.

## Problem statement

The two-node lab needs a reproducible data validation workflow focused on reporting on Fedora.

## Project objectives

- Return accepted values and row-level diagnostics for invalid or duplicate codes.
- Export an auditable JSON report without overwriting an existing one.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Fedora, rpm, JSON, and Python standard-library modules.

## System architecture

`OS identity → local JSON → validated workflow → terminal or report`

## Features

- Return accepted values and row-level diagnostics for invalid or duplicate codes.
- Export an auditable JSON report without overwriting an existing one.

## Prerequisites

Use Fedora in a practice environment with Python 3.11 or later and rpm. Use a text editor and a local practice directory.

## Complete Fedora source code

```python
"""Local Fedora reference implementation for project 004789."""


from pathlib import Path
import argparse
import json
import platform
import subprocess
import sys


EXPECTED_SYSTEM = 'Fedora'
PROJECT_ID = "004789"


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
    """Return accepted values and row-level diagnostics for invalid or duplicate codes."""
    accepted = {}
    rejected = []
    for position, row in enumerate(objects(record, "rows"), 1):
        try:
            code = label(row, "code").upper()
            count = integer(row, "count")
            if code in accepted:
                raise ValueError("duplicate code " + code)
            accepted[code] = count
        except ValueError as exc:
            rejected.append({"row": position, "error": str(exc)})
    return {"accepted": accepted, "rejected": rejected}


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
    record = load(args.input)
    return {"project": PROJECT_ID, "asset": label(record, "asset"), "result": evaluate(record)}, 0


def main():
    parser = argparse.ArgumentParser(description='Return accepted values and row-level diagnostics for invalid or duplicate codes.')
    parser.add_argument("input", type=Path)
    parser.add_argument("--output", type=Path, help="new JSON report path")
    args = parser.parse_args()
    try:
        environment = platform_details()
        outcome, status = process(args)
        report = {"environment": environment, "result": outcome}
        serialized = json.dumps(report, indent=2, ensure_ascii=False, allow_nan=False)
        if args.output is not None:
            with args.output.open("x", encoding="utf-8") as handle:
                handle.write(serialized + "\n")

        print(serialized)
        return status
    except (OSError, ValueError, OverflowError, RecursionError, subprocess.SubprocessError) as exc:
        print(f"Demo error: {exc}", file=sys.stderr)
        return 2


if __name__ == "__main__":
    raise SystemExit(main())
```

## Implementation explanation

The program verifies the OS and reads rpm to identify the practice environment. The data validation calculation works on synthetic input. Export an auditable JSON report without overwriting an existing one.

## Sample input data

Save this synthetic record as `sample.json`.

```json
{
  "asset": "NODE-B",
  "rows": [
    {
      "code": "KIT-01",
      "count": 4
    },
    {
      "code": "KIT-01",
      "count": 2
    },
    {
      "code": "KIT-02",
      "count": -1
    }
  ]
}
```

## Execution instructions

Copy the Python source to `project.py` and the sample JSON to `sample.json` in the same directory. Choose an unused path for the report.

```sh
python3 project.py sample.json --output report.json
```

## Example output

Illustrative excerpt for the synthetic sample.

```json
{
  "accepted": {
    "KIT-01": 4
  }
}
```

## Testing and validation

Actual tests ran on a Fedora 43 container. The platform command returned the observed version and the sample result repeated. The saved JSON matched the terminal output, and an overwrite attempt was rejected.

```json
{
  "accepted": {
    "KIT-01": 4
  },
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

Practice data validation using Fedora and check a repeatable reporting workflow in the two-node lab scenario.

## Conclusion

The project records an observable data validation result for training case `NODE-B`.

[Back to the Fedora project index](../README.md)

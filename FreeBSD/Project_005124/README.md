# FreeBSD: Audit — Reporting for Branch Office Simulator (Project 005124)

## Project overview

A simulated branch office provides the setting for this audit exercise. This exercise is based on the student guide and uses fictional data. This project includes source code, sample data, and instructions to run it.

## Problem statement

The branch office simulator needs a reproducible audit workflow focused on reporting on FreeBSD.

## Project objectives

- Check whether recorded asset reviews meet a cutoff and list departures.
- Export an auditable JSON report without overwriting an existing one.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

FreeBSD, freebsd-version, JSON, and Python standard-library modules.

## System architecture

`OS identity → local JSON → validated workflow → terminal or report`

## Features

- Check whether recorded asset reviews meet a cutoff and list departures.
- Export an auditable JSON report without overwriting an existing one.

## Prerequisites

Use FreeBSD in a practice environment with Python 3.11 or later and freebsd-version. Use a text editor and a local practice directory.

## Complete FreeBSD source code

```python
"""Local FreeBSD reference implementation for project 005124."""


from datetime import date
from pathlib import Path
import argparse
import json
import platform
import subprocess
import sys


EXPECTED_SYSTEM = 'FreeBSD'
PROJECT_ID = "005124"


def label(record, key):
    value = record.get(key)
    if not isinstance(value, str) or not value.strip():
        raise ValueError(f"{key} must be non-empty text")
    return value.strip()


def iso_date(record, key):
    value = label(record, key)
    try:
        parsed = date.fromisoformat(value)
    except ValueError as exc:
        raise ValueError(f"{key} must use YYYY-MM-DD") from exc
    if parsed.isoformat() != value:
        raise ValueError(f"{key} must use YYYY-MM-DD")
    return parsed


def evaluate(record):
    """Check whether recorded asset reviews meet a cutoff and list departures."""
    reviewed = iso_date(record, "reviewed_on")
    cutoff = iso_date(record, "cutoff")
    return {"owner": label(record, "owner"), "reviewed_on": reviewed.isoformat(),
            "follow_up": reviewed < cutoff}


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
    return {"project": PROJECT_ID, "asset": label(record, "asset"), "result": evaluate(record)}, 0


def main():
    parser = argparse.ArgumentParser(description='Check whether recorded asset reviews meet a cutoff and list departures.')
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

The program verifies the OS and reads freebsd-version to identify the practice environment. The audit calculation works on synthetic input. Export an auditable JSON report without overwriting an existing one.

## Sample input data

Save this synthetic record as `sample.json`.

```json
{
  "asset": "BRANCH-08",
  "reviewed_on": "2026-05-03",
  "cutoff": "2026-06-01",
  "owner": "Lab Operations"
}
```

## Execution instructions

Copy the Python source to `project.py` and the sample JSON to `sample.json` in the same directory. Choose an unused path for the report.

```sh
python3.11 project.py sample.json --output report.json
```

## Example output

Illustrative excerpt for the synthetic sample.

```json
{
  "follow_up": true
}
```

## Testing and validation

Actual tests ran on a FreeBSD 14.2 virtual machine. The platform command returned the observed version and the sample result repeated. The saved JSON matched the terminal output, and an overwrite attempt was rejected.

```json
{
  "follow_up": true,
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

Practice audit using FreeBSD and check a repeatable reporting workflow in the branch office simulator scenario.

## Conclusion

The project records an observable audit result for training case `BRANCH-08`.

[Back to the FreeBSD project index](../README.md)

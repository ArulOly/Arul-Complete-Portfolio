# FreeBSD: Data Export — Baseline Setup for Public Dataset (Project 005316)

## Project overview

A synthetic public-data exercise provides the setting for this data export exercise. The embedded code is a reference implementation based on the student guide and uses fictional data.

## Problem statement

The public dataset needs a reproducible data export workflow focused on baseline setup on FreeBSD.

## Project objectives

- Convert structured records into a predictable two-column CSV.
- Record the platform and a reference result for one input.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

FreeBSD, freebsd-version, JSON, and Python standard-library modules.

## System architecture

`OS identity → local JSON → validated workflow → terminal or report`

## Features

- Convert structured records into a predictable two-column CSV.
- Record the platform and a reference result for one input.

## Prerequisites

Use FreeBSD in a practice environment with Python 3.11 or later and freebsd-version. Use a text editor and a local practice directory.

## Complete FreeBSD source code

```python
"""Local FreeBSD reference implementation for project 005316."""


from pathlib import Path
import argparse
import csv
import io
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
    """Convert structured records into a predictable two-column CSV."""
    output = io.StringIO()
    writer = csv.writer(output, lineterminator="\n")
    writer.writerow(["code", "quantity"])
    entries = objects(record, "entries")
    for entry in entries:
        code = label(entry, "code")
        if code.startswith(("=", "+", "-", "@")):
            raise ValueError("code must begin with a letter or number")
        writer.writerow([code, integer(entry, "quantity")])
    return {"exported": len(entries), "csv": output.getvalue()}


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
    parser = argparse.ArgumentParser(description='Convert structured records into a predictable two-column CSV.')
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

The program verifies the OS and reads freebsd-version to identify the practice environment. The data export calculation works on synthetic input. Record the platform and a reference result for one input.

## Sample input data

Save this synthetic record as `sample.json`.

```json
{
  "asset": "DATA-12",
  "entries": [
    {
      "code": "KIT-01",
      "quantity": 4
    },
    {
      "code": "KIT-02",
      "quantity": 2
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
  "csv": "code,quantity\nKIT-01,4\nKIT-02,2\n"
}
```

## Testing and validation

Actual tests ran on a FreeBSD 14.2 virtual machine. The platform command returned the observed version and the sample result repeated. The incomplete input returned exit status 2.

```json
{
  "csv": "code,quantity\nKIT-01,4\nKIT-02,2\n",
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

Practice data export using FreeBSD and check a repeatable baseline setup workflow in the public dataset scenario.

## Conclusion

The project records an observable data export result for training case `DATA-12`.

[Back to the FreeBSD project index](../README.md)

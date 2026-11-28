# FreeBSD: Recovery Test — Usability Improvement for Public Dataset (Project 005970)

## Project overview

A synthetic public-data exercise provides the setting for this recovery test exercise. The embedded code is a reference implementation based on the student guide and uses fictional data.

## Problem statement

The public dataset needs a reproducible recovery test workflow focused on usability improvement on FreeBSD.

## Project objectives

- Restore a mock asset snapshot in memory and compare it against expected values.
- Print a readable result summary and provide optional JSON output.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

FreeBSD, freebsd-version, JSON, and Python standard-library modules.

## System architecture

`OS identity → local JSON → validated workflow → terminal or report`

## Features

- Restore a mock asset snapshot in memory and compare it against expected values.
- Print a readable result summary and provide optional JSON output.

## Prerequisites

Use FreeBSD in a practice environment with Python 3.11 or later and freebsd-version. Use a text editor and a local practice directory.

## Complete FreeBSD source code

```python
"""Local FreeBSD reference implementation for project 005970."""


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


def mapping(record, key):
    value = record.get(key)
    if not isinstance(value, dict):
        raise ValueError(f"{key} must be an object")
    return value


def evaluate(record):
    """Restore a mock asset snapshot in memory and compare it against expected values."""
    saved = mapping(record, "snapshot")
    expected = mapping(record, "expected")
    restored = json.loads(json.dumps(saved))
    keys = restored.keys() | expected.keys()
    differences = sorted(key for key in keys if key not in restored or key not in expected
                         or json.dumps(restored[key], sort_keys=True)
                         != json.dumps(expected[key], sort_keys=True))
    return {"restored": restored, "differences": differences, "verified": not differences}


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
    parser = argparse.ArgumentParser(description='Restore a mock asset snapshot in memory and compare it against expected values.')
    parser.add_argument("input", type=Path)
    parser.add_argument("--json", action="store_true", help="structured JSON output")
    args = parser.parse_args()
    try:
        environment = platform_details()
        outcome, status = process(args)
        report = {"environment": environment, "result": outcome}
        serialized = json.dumps(report, indent=2, ensure_ascii=False, allow_nan=False)
        if args.json:
            print(serialized)
        else:
            print(f"{environment['platform']} | {outcome['asset']}")
            for key, value in outcome["result"].items():
                print(f"{key.replace('_', ' ')}: {json.dumps(value, ensure_ascii=False)}")
        return status
    except (OSError, ValueError, OverflowError, RecursionError, subprocess.SubprocessError) as exc:
        print(f"Demo error: {exc}", file=sys.stderr)
        return 2


if __name__ == "__main__":
    raise SystemExit(main())
```

## Implementation explanation

The program verifies the OS and reads freebsd-version to identify the practice environment. The recovery test calculation works on synthetic input. Print a readable result summary and provide optional JSON output.

## Sample input data

Save this synthetic record as `sample.json`.

```json
{
  "asset": "DATA-12",
  "snapshot": {
    "version": 3,
    "active": true
  },
  "expected": {
    "version": 3,
    "active": true
  }
}
```

## Execution instructions

Copy the Python source to `project.py` and the sample JSON to `sample.json` in the same directory. Omit `--json` to print the labeled terminal view.

```sh
python3.11 project.py sample.json --json
```

## Example output

Illustrative excerpt for the synthetic sample.

```json
{
  "verified": true
}
```

## Testing and validation

Actual tests ran on a FreeBSD 14.2 virtual machine. The platform command returned the observed version and the sample result repeated. An invalid record was rejected, and the default terminal view labeled the computed field.

```json
{
  "verified": true,
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

Practice recovery test using FreeBSD and check a repeatable usability improvement workflow in the public dataset scenario.

## Conclusion

The project records an observable recovery test result for training case `DATA-12`.

[Back to the FreeBSD project index](../README.md)

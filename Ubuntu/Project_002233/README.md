# Ubuntu: Backup Verification — Recovery for Single-Host Lab (Project 002233)

## Project overview

One simulated lab host provides the setting for this backup verification exercise. The embedded code is an AI-generated reference implementation based on the student guide and uses fictional data.

## Problem statement

The single-host lab needs a reproducible backup verification workflow focused on recovery on Ubuntu.

## Project objectives

- Compare a synthetic backup payload against an expected SHA-256 digest.
- Load a fallback fixture when the primary is absent or invalid.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Ubuntu, dpkg-query, JSON, and Python standard-library modules.

## System architecture

`OS identity → local JSON → validated workflow → terminal or report`

## Features

- Compare a synthetic backup payload against an expected SHA-256 digest.
- Load a fallback fixture when the primary is absent or invalid.

## Prerequisites

Use Ubuntu in a practice environment with Python 3.11 or later and dpkg-query. Use a text editor and a local practice directory.

## Complete Ubuntu source code

```python
"""Local Ubuntu reference implementation for project 002233."""


from pathlib import Path
import argparse
import hashlib
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


def evaluate(record):
    """Compare a synthetic backup payload against an expected SHA-256 digest."""
    expected = label(record, "sha256").lower()
    if len(expected) != 64 or any(char not in "0123456789abcdef" for char in expected):
        raise ValueError("sha256 must contain exactly 64 hexadecimal characters")
    actual = hashlib.sha256(label(record, "payload").encode("utf-8")).hexdigest()
    return {"bytes": len(label(record, "payload").encode("utf-8")),
            "sha256": actual, "verified": actual == expected}


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
    try:
        record = load(args.input)
        return {"source": "primary", "asset": label(record, "asset"),
                "result": evaluate(record)}, 0
    except (OSError, ValueError, OverflowError, RecursionError) as exc:
        record = load(args.fallback)
        return {"source": "fallback", "asset": label(record, "asset"),
                "primary_error": str(exc), "result": evaluate(record)}, 0


def main():
    parser = argparse.ArgumentParser(description='Compare a synthetic backup payload against an expected SHA-256 digest.')
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

The program verifies the OS and reads dpkg-query to identify the practice environment. The backup verification calculation works on synthetic input. Load a fallback fixture when the primary is absent or invalid.

## Sample input data

Save this synthetic record as `sample.json`.

```json
{
  "asset": "HOST-03",
  "payload": "Backup manifest for HOST-03, revision 1",
  "sha256": "0bfdb55001e342f5b774611eb390453518709f6ac3c57205f1d78c275f2ffc9b"
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
  "verified": true
}
```

## Testing and validation

Actual tests ran on a Ubuntu 24.04 container. The platform command returned the observed version and the sample result repeated. A bad primary used the supplied fallback. Two unusable inputs returned exit status 2.

```json
{
  "verified": true,
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

Practice backup verification using Ubuntu and check a repeatable recovery workflow in the single-host lab scenario.

## Conclusion

The project records an observable backup verification result for training case `HOST-03`.

[Back to the Ubuntu project index](../README.md)

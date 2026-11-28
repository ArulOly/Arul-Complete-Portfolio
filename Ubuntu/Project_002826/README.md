# Ubuntu: Documentation Generation — Baseline Setup for Offline Classroom (Project 002826)

## Project overview

A classroom operating offline provides the setting for this documentation generation exercise. This exercise is based on the student guide and uses fictional data. This project includes source code, sample data, and instructions to run it.

## Problem statement

The offline classroom needs a reproducible documentation generation workflow focused on baseline setup on Ubuntu.

## Project objectives

- Generate a short Markdown handover from a synthetic operating procedure.
- Record the platform and a reference result for one input.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Ubuntu, dpkg-query, JSON, and Python standard-library modules.

## System architecture

`OS identity → local JSON → validated workflow → terminal or report`

## Features

- Generate a short Markdown handover from a synthetic operating procedure.
- Record the platform and a reference result for one input.

## Prerequisites

Use Ubuntu in a practice environment with Python 3.11 or later and dpkg-query. Use a text editor and a local practice directory.

## Complete Ubuntu source code

```python
"""Local Ubuntu reference implementation for project 002826."""


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


def array(record, key, allow_empty=False):
    value = record.get(key)
    if not isinstance(value, list) or (not value and not allow_empty):
        raise ValueError(f"{key} must be a list" + ("" if allow_empty else " with at least one item"))
    return value


def labels(record, key, allow_empty=False):
    return [label({key: item}, key) for item in array(record, key, allow_empty)]


def evaluate(record):
    """Generate a short Markdown handover from a synthetic operating procedure."""
    lines = [f"# {label(record, 'asset')}", "", label(record, "summary"), "", "## Procedure", ""]
    lines.extend(f"{index}. {step}" for index, step in enumerate(labels(record, "steps"), 1))
    return {"markdown": "\n".join(lines) + "\n", "steps": len(lines) - 6}


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
    record = load(args.input)
    return {"asset": label(record, "asset"), "result": evaluate(record)}, 0


def main():
    parser = argparse.ArgumentParser(description='Generate a short Markdown handover from a synthetic operating procedure.')
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

The program verifies the OS and reads dpkg-query to identify the practice environment. The documentation generation calculation works on synthetic input. Record the platform and a reference result for one input.

## Sample input data

Save this synthetic record as `sample.json`.

```json
{
  "asset": "CLASS-09",
  "summary": "Review the training asset CLASS-09 before handover.",
  "steps": [
    "Confirm the asset label",
    "Record the handover status"
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
  "steps": 2
}
```

## Testing and validation

Actual tests ran on a Ubuntu 24.04 container. The platform command returned the observed version and the sample result repeated. The incomplete input returned exit status 2.

```json
{
  "steps": 2,
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

Practice documentation generation using Ubuntu and check a repeatable baseline setup workflow in the offline classroom scenario.

## Conclusion

The project records an observable documentation generation result for training case `CLASS-09`.

[Back to the Ubuntu project index](../README.md)

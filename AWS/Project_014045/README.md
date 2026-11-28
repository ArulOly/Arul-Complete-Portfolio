# AWS: Inventory — Usability Improvement for Help Desk (Project 014045)

## Project overview

A help desk training queue provides the setting for this inventory exercise. The embedded code is a reference implementation based on the student guide and uses fictional data.

## Problem statement

The help desk needs a reproducible inventory workflow for an event modeled on Amazon S3.

## Project objectives

- Count items and identify stock below the reorder level.
- Print a readable result summary and provide optional JSON output.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Amazon S3 event format, a simulated local object store, JSON, and Python standard-library modules.

## System architecture

`Synthetic event → object address → local fixture lookup → validated workflow → report`

## Features

- Count items and identify stock below the reorder level.
- Print a readable result summary and provide optional JSON output.

## Prerequisites

Use Python 3.10 or later. The offline practice run requires no cloud account or packages. Use a text editor and a local practice directory.

## Complete AWS source code

```python
"""Local AWS reference implementation for project 014045."""


from pathlib import Path
from urllib.parse import unquote_plus
import argparse
import json
import sys




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


def evaluate(record):
    """Count items and identify stock below the reorder level."""
    stock = integer(record, "stock")
    reorder = integer(record, "reorder_at", minimum=1)
    return {"stock": stock, "reorder_at": reorder, "needs_reorder": stock < reorder}


def platform_details():
    return {"platform": 'AWS', "service": 'Amazon S3', "mode": "offline fixture"}


def object_address(event):
    notifications = event["Records"]
    if not isinstance(notifications, list) or len(notifications) != 1:
        raise ValueError("Expected one S3 notification record")
    notice = notifications[0]
    if notice["eventSource"] != "aws:s3" or not notice["eventName"].startswith("ObjectCreated:"):
        raise ValueError("Expected an S3 object-created event")
    bucket = notice["s3"]["bucket"]["name"]
    key = unquote_plus(notice["s3"]["object"]["key"])
    if not bucket or not key:
        raise ValueError("Expected a bucket and object key")
    return f"s3://{bucket}/{key}"


def load(path):
    fixture = json.loads(path.read_text(encoding="utf-8"))
    if not isinstance(fixture, dict):
        raise ValueError("Expected a JSON fixture object")
    try:
        address = object_address(fixture["event"])
        value = fixture["objects"][address]
    except (KeyError, IndexError, TypeError, AttributeError) as exc:
        raise ValueError("Invalid event or event object missing from local storage") from exc
    if not isinstance(value, dict):
        raise ValueError("Expected an object payload")
    json.dumps(value, allow_nan=False)
    return value


def process(args):
    record = load(args.input)
    return {"asset": label(record, "asset"), "result": evaluate(record)}, 0


def main():
    parser = argparse.ArgumentParser(description='Count items and identify stock below the reorder level.')
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
    except (OSError, ValueError, OverflowError, RecursionError) as exc:
        print(f"Demo error: {exc}", file=sys.stderr)
        return 2


if __name__ == "__main__":
    raise SystemExit(main())
```

## Implementation explanation

The adapter reads the S3 event notification and extracts an object address. It uses that address to read from a local dictionary. The inventory workflow then evaluates the selected payload.

## Sample input data

Save the synthetic event and object fixture together as `sample.json`.

```json
{
  "event": {
    "Records": [
      {
        "eventSource": "aws:s3",
        "eventName": "ObjectCreated:Put",
        "s3": {
          "bucket": {
            "name": "portfolio-example"
          },
          "object": {
            "key": "inputs/portfolio+014045.json"
          }
        }
      }
    ]
  },
  "objects": {
    "s3://portfolio-example/inputs/portfolio 014045.json": {
      "asset": "DESK-14",
      "stock": 2,
      "reorder_at": 4
    }
  }
}
```

## Execution instructions

Save the source as `project.py` alongside `sample.json`. Omit `--json` to see a compact summary.

```sh
python project.py sample.json --json
```

## Example output

Illustrative excerpt for the synthetic sample.

```json
{
  "needs_reorder": true
}
```

## Testing and validation

Actual tests ran the Python program against a local simulated object store. The sample field matched the expected value and repeated. An invalid record was rejected, and the default terminal view labeled the computed field.

```json
{
  "needs_reorder": true
}
```

## Error handling

The adapter raises a diagnostic when the event is malformed or its object is missing from the fixture. Invalid payloads follow the documented CLI, batch, or recovery behavior.

## Troubleshooting

- Confirm the documented runtime and copy the complete source block.
- Start with the documented sample and use the filename or request path shown above.
- Compare the observed value with the example before changing an input.

## Limitations

This offline reference uses synthetic event data and a local dictionary. Cloud deployment, credentials, permissions, and live event delivery are outside the tested scope.

## Learning outcomes

Practice inventory using AWS and check a repeatable usability improvement workflow in the help desk scenario.

## Conclusion

The project records an observable inventory result for training case `DESK-14`.

[Back to the AWS project index](../README.md)

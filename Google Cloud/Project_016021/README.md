# Google Cloud: Inventory — Baseline Setup for Branch Office Simulator (Project 016021)

## Project overview

A simulated branch office provides the setting for this inventory exercise. This exercise is based on the student guide and uses fictional data. This project includes source code, sample data, and instructions to run it.

## Problem statement

The branch office simulator needs a reproducible inventory workflow for an event modeled on Google Cloud Storage.

## Project objectives

- Count items and identify stock below the reorder level.
- Record the platform and a reference result for one input.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Google Cloud Storage event format, a simulated local object store, JSON, and Python standard-library modules.

## System architecture

`Synthetic event → object address → local fixture lookup → validated workflow → report`

## Features

- Count items and identify stock below the reorder level.
- Record the platform and a reference result for one input.

## Prerequisites

Use Python 3.10 or later. The offline practice run requires no cloud account or packages. Use a text editor and a local practice directory.

## Complete Google Cloud source code

```python
"""Local Google Cloud reference implementation for project 016021."""


from pathlib import Path
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
    return {"platform": 'Google Cloud', "service": 'Google Cloud Storage', "mode": "offline fixture"}


def object_address(event):
    if event["specversion"] != "1.0" or event["type"] != "google.cloud.storage.object.v1.finalized":
        raise ValueError("Expected a CloudEvents 1.0 object-finalized event")
    data = event["data"]
    bucket, key = data["bucket"], data["name"]
    if not bucket or not key:
        raise ValueError("Expected a bucket and object name")
    return f"gs://{bucket}/{key}"


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
    args = parser.parse_args()
    try:
        environment = platform_details()
        outcome, status = process(args)
        report = {"environment": environment, "result": outcome}
        serialized = json.dumps(report, indent=2, ensure_ascii=False, allow_nan=False)
        print(serialized)
        return status
    except (OSError, ValueError, OverflowError, RecursionError) as exc:
        print(f"Demo error: {exc}", file=sys.stderr)
        return 2


if __name__ == "__main__":
    raise SystemExit(main())
```

## Implementation explanation

The adapter reads the Cloud Storage object-finalized CloudEvent and extracts an object address. It uses that address to read from a local dictionary. The inventory workflow then evaluates the selected payload.

## Sample input data

Save the synthetic event and object fixture together as `sample.json`.

```json
{
  "event": {
    "specversion": "1.0",
    "type": "google.cloud.storage.object.v1.finalized",
    "id": "016021",
    "source": "//storage.googleapis.com/projects/_/buckets/portfolio-example",
    "data": {
      "bucket": "portfolio-example",
      "name": "inputs/portfolio 016021.json"
    }
  },
  "objects": {
    "gs://portfolio-example/inputs/portfolio 016021.json": {
      "asset": "BRANCH-08",
      "stock": 2,
      "reorder_at": 4
    }
  }
}
```

## Execution instructions

Save the source as `project.py` alongside `sample.json`.

```sh
python project.py sample.json
```

## Example output

Illustrative excerpt for the synthetic sample.

```json
{
  "needs_reorder": true
}
```

## Testing and validation

Actual tests ran the Python program against a local simulated object store. The sample field matched the expected value and repeated. The incomplete input returned exit status 2.

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

Practice inventory using Google Cloud and check a repeatable baseline setup workflow in the branch office simulator scenario.

## Conclusion

The project records an observable inventory result for training case `BRANCH-08`.

[Back to the Google Cloud project index](../README.md)

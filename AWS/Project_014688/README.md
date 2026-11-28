# AWS: Configuration Comparison — Recovery for Two-Node Lab (Project 014688)

## Project overview

A two-node lab simulation provides the setting for this configuration comparison exercise. The embedded code is an AI-generated reference implementation based on the student guide and uses fictional data.

## Problem statement

The two-node lab needs a reproducible configuration comparison workflow for an event modeled on Amazon S3.

## Project objectives

- Identify added, removed, and changed settings between two configuration objects.
- Load a fallback fixture when the primary is absent or invalid.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Amazon S3 event format, a simulated local object store, JSON, and Python standard-library modules.

## System architecture

`Synthetic event → object address → local fixture lookup → validated workflow → report`

## Features

- Identify added, removed, and changed settings between two configuration objects.
- Load a fallback fixture when the primary is absent or invalid.

## Prerequisites

Use Python 3.10 or later. The offline practice run requires no cloud account or packages. Use a text editor and a local practice directory.

## Complete AWS source code

```python
"""Local AWS reference implementation for project 014688."""


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
    try:
        record = load(args.input)
        return {"source": "primary", "asset": label(record, "asset"),
                "result": evaluate(record)}, 0
    except (OSError, ValueError, OverflowError, RecursionError) as exc:
        record = load(args.fallback)
        return {"source": "fallback", "asset": label(record, "asset"),
                "primary_error": str(exc), "result": evaluate(record)}, 0


def main():
    parser = argparse.ArgumentParser(description='Identify added, removed, and changed settings between two configuration objects.')
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
    except (OSError, ValueError, OverflowError, RecursionError) as exc:
        print(f"Demo error: {exc}", file=sys.stderr)
        return 2


if __name__ == "__main__":
    raise SystemExit(main())
```

## Implementation explanation

The adapter reads the S3 event notification and extracts an object address. It uses that address to read from a local dictionary. The configuration comparison workflow then evaluates the selected payload.

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
            "key": "inputs/portfolio+014688.json"
          }
        }
      }
    ]
  },
  "objects": {
    "s3://portfolio-example/inputs/portfolio 014688.json": {
      "asset": "NODE-B",
      "baseline": {
        "timeout": 30,
        "theme": "light"
      },
      "current": {
        "timeout": 45,
        "retries": 2
      }
    }
  }
}
```

## Execution instructions

Save the source as `project.py` alongside `sample.json`. Save a second copy of the sample as `fallback.json`.

```sh
python project.py sample.json --fallback fallback.json
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

Actual tests ran the Python program against a local simulated object store. The sample field matched the expected value and repeated. A bad primary used the supplied fallback. Two unusable inputs returned exit status 2.

```json
{
  "changed": [
    "timeout"
  ]
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

Practice configuration comparison using AWS and check a repeatable recovery workflow in the two-node lab scenario.

## Conclusion

The project records an observable configuration comparison result for training case `NODE-B`.

[Back to the AWS project index](../README.md)

# Google Cloud: Monitoring — Reporting for Branch Office Simulator (Project 016074)

## Project overview

A simulated branch office provides the setting for this monitoring exercise. The embedded code is a reference implementation based on the student guide and uses fictional data.

## Problem statement

The branch office simulator needs a reproducible monitoring workflow for an event modeled on Google Cloud Storage.

## Project objectives

- Summarize utilization readings and count readings above the limit.
- Export an auditable JSON report without overwriting an existing one.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Google Cloud Storage event format, a simulated local object store, JSON, and Python standard-library modules.

## System architecture

`Synthetic event → object address → local fixture lookup → validated workflow → report`

## Features

- Summarize utilization readings and count readings above the limit.
- Export an auditable JSON report without overwriting an existing one.

## Prerequisites

Use Python 3.10 or later. The offline practice run requires no cloud account or packages. Use a text editor and a local practice directory.

## Complete Google Cloud source code

```python
"""Local Google Cloud reference implementation for project 016074."""


from pathlib import Path
import argparse
import json
import statistics
import sys


PROJECT_ID = "016074"


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


def series(record, key, maximum=1_000_000_000):
    return [integer({key: item}, key, maximum=maximum) for item in array(record, key)]


def evaluate(record):
    """Summarize utilization readings and count readings above the limit."""
    readings = series(record, "readings_pct", maximum=100)
    limit = integer(record, "limit_pct", maximum=100)
    return {"mean_pct": round(statistics.mean(readings), 2),
            "peak_pct": max(readings), "above_limit": sum(n > limit for n in readings)}


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
    return {"project": PROJECT_ID, "asset": label(record, "asset"), "result": evaluate(record)}, 0


def main():
    parser = argparse.ArgumentParser(description='Summarize utilization readings and count readings above the limit.')
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
    except (OSError, ValueError, OverflowError, RecursionError) as exc:
        print(f"Demo error: {exc}", file=sys.stderr)
        return 2


if __name__ == "__main__":
    raise SystemExit(main())
```

## Implementation explanation

The adapter reads the Cloud Storage object-finalized CloudEvent and extracts an object address. It uses that address to read from a local dictionary. The monitoring workflow then evaluates the selected payload.

## Sample input data

Save the synthetic event and object fixture together as `sample.json`.

```json
{
  "event": {
    "specversion": "1.0",
    "type": "google.cloud.storage.object.v1.finalized",
    "id": "016074",
    "source": "//storage.googleapis.com/projects/_/buckets/portfolio-example",
    "data": {
      "bucket": "portfolio-example",
      "name": "inputs/portfolio 016074.json"
    }
  },
  "objects": {
    "gs://portfolio-example/inputs/portfolio 016074.json": {
      "asset": "BRANCH-08",
      "readings_pct": [
        35,
        81,
        54
      ],
      "limit_pct": 80
    }
  }
}
```

## Execution instructions

Save the source as `project.py` alongside `sample.json`. Use a new report path for each export.

```sh
python project.py sample.json --output report.json
```

## Example output

Illustrative excerpt for the synthetic sample.

```json
{
  "above_limit": 1
}
```

## Testing and validation

Actual tests ran the Python program against a local simulated object store. The sample field matched the expected value and repeated. The saved JSON matched the terminal output, and an overwrite attempt was rejected.

```json
{
  "above_limit": 1
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

Practice monitoring using Google Cloud and check a repeatable reporting workflow in the branch office simulator scenario.

## Conclusion

The project records an observable monitoring result for training case `BRANCH-08`.

[Back to the Google Cloud project index](../README.md)

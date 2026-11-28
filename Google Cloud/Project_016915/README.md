# Google Cloud: Visual Reporting — Usability Improvement for Home Development (Project 016915)

## Project overview

A home development workspace provides the setting for this visual reporting exercise. This exercise is based on the student guide and uses fictional data. This project includes source code, sample data, and instructions to run it.

## Problem statement

The home development needs a reproducible visual reporting workflow for an event modeled on Google Cloud Storage.

## Project objectives

- Create a proportional text bar chart with exact values and a labeled scale.
- Print a readable result summary and provide optional JSON output.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Google Cloud Storage event format, a simulated local object store, JSON, and Python standard-library modules.

## System architecture

`Synthetic event → object address → local fixture lookup → validated workflow → report`

## Features

- Create a proportional text bar chart with exact values and a labeled scale.
- Print a readable result summary and provide optional JSON output.

## Prerequisites

Use Python 3.10 or later. The offline practice run requires no cloud account or packages. Use a text editor and a local practice directory.

## Complete Google Cloud source code

```python
"""Local Google Cloud reference implementation for project 016915."""


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


def mapping(record, key):
    value = record.get(key)
    if not isinstance(value, dict):
        raise ValueError(f"{key} must be an object")
    return value


def evaluate(record):
    """Create a proportional text bar chart with exact values and a labeled scale."""
    values = mapping(record, "values")
    if not values:
        raise ValueError("values must contain at least one category")
    counts = {key: integer(values, key) for key in values}
    width = integer(record, "width", minimum=1, maximum=60)
    peak = max(counts.values())
    bars = [f"{key}: {'#' * ((value * width + peak // 2) // peak if peak else 0)} ({value})"
            for key, value in sorted(counts.items())]
    return {"peak": peak, "max_bar_width": width, "bars": bars}


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
    parser = argparse.ArgumentParser(description='Create a proportional text bar chart with exact values and a labeled scale.')
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

The adapter reads the Cloud Storage object-finalized CloudEvent and extracts an object address. It uses that address to read from a local dictionary. The visual reporting workflow then evaluates the selected payload.

## Sample input data

Save the synthetic event and object fixture together as `sample.json`.

```json
{
  "event": {
    "specversion": "1.0",
    "type": "google.cloud.storage.object.v1.finalized",
    "id": "016915",
    "source": "//storage.googleapis.com/projects/_/buckets/portfolio-example",
    "data": {
      "bucket": "portfolio-example",
      "name": "inputs/portfolio 016915.json"
    }
  },
  "objects": {
    "gs://portfolio-example/inputs/portfolio 016915.json": {
      "asset": "DEV-05",
      "values": {
        "available": 12,
        "reserved": 7,
        "repair": 3
      },
      "width": 12
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
  "peak": 12
}
```

## Testing and validation

Actual tests ran the Python program against a local simulated object store. The sample field matched the expected value and repeated. An invalid record was rejected, and the default terminal view labeled the computed field.

```json
{
  "peak": 12
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

Practice visual reporting using Google Cloud and check a repeatable usability improvement workflow in the home development scenario.

## Conclusion

The project records an observable visual reporting result for training case `DEV-05`.

[Back to the Google Cloud project index](../README.md)

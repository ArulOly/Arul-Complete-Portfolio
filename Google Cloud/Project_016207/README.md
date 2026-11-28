# Google Cloud: Backup Verification — Automation for Small Business (Project 016207)

## Project overview

Local shop operations provides the setting for this backup verification exercise. The embedded code is a reference implementation based on the student guide and uses fictional data.

## Problem statement

The small business needs a reproducible backup verification workflow for an event modeled on Google Cloud Storage.

## Project objectives

- Compare a synthetic backup payload against an expected SHA-256 digest.
- Process a batch and keep successful rows alongside diagnostics.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Google Cloud Storage event format, a simulated local object store, JSON, and Python standard-library modules.

## System architecture

`Synthetic event → object address → local fixture lookup → validated workflow → report`

## Features

- Compare a synthetic backup payload against an expected SHA-256 digest.
- Process a batch and keep successful rows alongside diagnostics.

## Prerequisites

Use Python 3.10 or later. The offline practice run requires no cloud account or packages. Use a text editor and a local practice directory.

## Complete Google Cloud source code

```python
"""Local Google Cloud reference implementation for project 016207."""


from pathlib import Path
import argparse
import hashlib
import json
import sys




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


def evaluate(record):
    """Compare a synthetic backup payload against an expected SHA-256 digest."""
    expected = label(record, "sha256").lower()
    if len(expected) != 64 or any(char not in "0123456789abcdef" for char in expected):
        raise ValueError("sha256 must contain exactly 64 hexadecimal characters")
    actual = hashlib.sha256(label(record, "payload").encode("utf-8")).hexdigest()
    return {"bytes": len(label(record, "payload").encode("utf-8")),
            "sha256": actual, "verified": actual == expected}


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
    records = array(load(args.input), "records")
    rows = []
    for index, candidate in enumerate(records, 1):
        try:
            if not isinstance(candidate, dict):
                raise ValueError("Expected a record")
            rows.append({"row": index, "asset": label(candidate, "asset"), "result": evaluate(candidate)})
        except (ValueError, OverflowError) as exc:
            rows.append({"row": index, "error": str(exc)})
    return {"rows": rows}, int(any("error" in row for row in rows))


def main():
    parser = argparse.ArgumentParser(description='Compare a synthetic backup payload against an expected SHA-256 digest.')
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

The adapter reads the Cloud Storage object-finalized CloudEvent and extracts an object address. It uses that address to read from a local dictionary. The backup verification workflow then evaluates the selected payload.

## Sample input data

Save the synthetic event and object fixture together as `sample.json`.

```json
{
  "event": {
    "specversion": "1.0",
    "type": "google.cloud.storage.object.v1.finalized",
    "id": "016207",
    "source": "//storage.googleapis.com/projects/_/buckets/portfolio-example",
    "data": {
      "bucket": "portfolio-example",
      "name": "inputs/portfolio 016207.json"
    }
  },
  "objects": {
    "gs://portfolio-example/inputs/portfolio 016207.json": {
      "records": [
        {
          "asset": "SHOP-22",
          "payload": "Backup manifest for SHOP-22, revision 1",
          "sha256": "89b1aa104bcff1c49c73ac1b9038613f2a2a26cacb23cafad20b419621ba8cde"
        },
        {
          "asset": "SHOP-22-B",
          "payload": "Backup manifest for SHOP-22, revision 1",
          "sha256": "89b1aa104bcff1c49c73ac1b9038613f2a2a26cacb23cafad20b419621ba8cde"
        }
      ]
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
  "verified": true
}
```

## Testing and validation

Actual tests ran the Python program against a local simulated object store. The sample field matched the expected value and repeated. A mixed batch preserved its successful row and returned exit status 1.

```json
{
  "verified": true
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

Practice backup verification using Google Cloud and check a repeatable automation workflow in the small business scenario.

## Conclusion

The project records an observable backup verification result for training case `SHOP-22`.

[Back to the Google Cloud project index](../README.md)

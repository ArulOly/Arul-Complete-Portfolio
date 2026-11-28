# Azure: Data Export — Automation for Offline Classroom (Project 015327)

## Project overview

A classroom operating offline provides the setting for this data export exercise. This exercise is based on the student guide and uses fictional data. This project includes source code, sample data, and instructions to run it.

## Problem statement

The offline classroom needs a reproducible data export workflow for an event modeled on Azure Blob Storage.

## Project objectives

- Convert structured records into a predictable two-column CSV.
- Process a batch and keep successful rows alongside diagnostics.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Azure Blob Storage event format, a simulated local object store, JSON, and Python standard-library modules.

## System architecture

`Synthetic event → object address → local fixture lookup → validated workflow → report`

## Features

- Convert structured records into a predictable two-column CSV.
- Process a batch and keep successful rows alongside diagnostics.

## Prerequisites

Use Python 3.10 or later. The offline practice run requires no cloud account or packages. Use a text editor and a local practice directory.

## Complete Azure source code

```python
"""Local Azure reference implementation for project 015327."""


from pathlib import Path
from urllib.parse import unquote, urlsplit
import argparse
import csv
import io
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
    return {"platform": 'Azure', "service": 'Azure Blob Storage', "mode": "offline fixture"}


def object_address(event):
    if event["eventType"] != "Microsoft.Storage.BlobCreated":
        raise ValueError("Expected a BlobCreated event")
    address = urlsplit(event["data"]["url"])
    if address.scheme != "https" or not address.hostname or not address.hostname.endswith(".blob.core.windows.net"):
        raise ValueError("Expected an Azure Blob Storage URL")
    segments = unquote(address.path).strip("/").split("/", 1)
    if len(segments) != 2 or not all(segments):
        raise ValueError("Expected a container and blob name")
    return f"https://{address.hostname}/" + "/".join(segments)


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
    except (OSError, ValueError, OverflowError, RecursionError) as exc:
        print(f"Demo error: {exc}", file=sys.stderr)
        return 2


if __name__ == "__main__":
    raise SystemExit(main())
```

## Implementation explanation

The adapter reads the Event Grid BlobCreated notification and extracts an object address. It uses that address to read from a local dictionary. The data export workflow then evaluates the selected payload.

## Sample input data

Save the synthetic event and object fixture together as `sample.json`.

```json
{
  "event": {
    "eventType": "Microsoft.Storage.BlobCreated",
    "data": {
      "url": "https://portfolioexample.blob.core.windows.net/practice/inputs/portfolio%20015327.json"
    }
  },
  "objects": {
    "https://portfolioexample.blob.core.windows.net/practice/inputs/portfolio 015327.json": {
      "records": [
        {
          "asset": "CLASS-09",
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
        },
        {
          "asset": "CLASS-09-B",
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
  "csv": "code,quantity\nKIT-01,4\nKIT-02,2\n"
}
```

## Testing and validation

Actual tests ran the Python program against a local simulated object store. The sample field matched the expected value and repeated. A mixed batch preserved its successful row and returned exit status 1.

```json
{
  "csv": "code,quantity\nKIT-01,4\nKIT-02,2\n"
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

Practice data export using Azure and check a repeatable automation workflow in the offline classroom scenario.

## Conclusion

The project records an observable data export result for training case `CLASS-09`.

[Back to the Azure project index](../README.md)

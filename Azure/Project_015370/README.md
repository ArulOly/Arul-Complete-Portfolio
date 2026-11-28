# Azure: Dashboard — Usability Improvement for Public Dataset (Project 015370)

## Project overview

A synthetic public-data exercise provides the setting for this dashboard exercise. This exercise is based on the student guide and uses fictional data. This project includes source code, sample data, and instructions to run it.

## Problem statement

The public dataset needs a reproducible dashboard workflow for an event modeled on Azure Blob Storage.

## Project objectives

- Aggregate ticket counts and show open workload as a share of all tickets.
- Print a readable result summary and provide optional JSON output.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Azure Blob Storage event format, a simulated local object store, JSON, and Python standard-library modules.

## System architecture

`Synthetic event → object address → local fixture lookup → validated workflow → report`

## Features

- Aggregate ticket counts and show open workload as a share of all tickets.
- Print a readable result summary and provide optional JSON output.

## Prerequisites

Use Python 3.10 or later. The offline practice run requires no cloud account or packages. Use a text editor and a local practice directory.

## Complete Azure source code

```python
"""Local Azure reference implementation for project 015370."""


from pathlib import Path
from urllib.parse import unquote, urlsplit
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
    """Aggregate ticket counts and show open workload as a share of all tickets."""
    queue = mapping(record, "queue")
    counts = {key: integer(queue, key) for key in ("open", "in_progress", "closed")}
    total = sum(counts.values())
    active = counts["open"] + counts["in_progress"]
    return {"counts": counts, "total": total,
            "active_pct": round(100 * active / total, 1) if total else 0.0}


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
    record = load(args.input)
    return {"asset": label(record, "asset"), "result": evaluate(record)}, 0


def main():
    parser = argparse.ArgumentParser(description='Aggregate ticket counts and show open workload as a share of all tickets.')
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

The adapter reads the Event Grid BlobCreated notification and extracts an object address. It uses that address to read from a local dictionary. The dashboard workflow then evaluates the selected payload.

## Sample input data

Save the synthetic event and object fixture together as `sample.json`.

```json
{
  "event": {
    "eventType": "Microsoft.Storage.BlobCreated",
    "data": {
      "url": "https://portfolioexample.blob.core.windows.net/practice/inputs/portfolio%20015370.json"
    }
  },
  "objects": {
    "https://portfolioexample.blob.core.windows.net/practice/inputs/portfolio 015370.json": {
      "asset": "DATA-12",
      "queue": {
        "open": 5,
        "in_progress": 3,
        "closed": 12
      }
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
  "active_pct": 40.0
}
```

## Testing and validation

Actual tests ran the Python program against a local simulated object store. The sample field matched the expected value and repeated. An invalid record was rejected, and the default terminal view labeled the computed field.

```json
{
  "active_pct": 40.0
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

Practice dashboard using Azure and check a repeatable usability improvement workflow in the public dataset scenario.

## Conclusion

The project records an observable dashboard result for training case `DATA-12`.

[Back to the Azure project index](../README.md)

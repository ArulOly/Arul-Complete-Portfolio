# Azure: Log Analysis — Reporting for Home Development (Project 015164)

## Project overview

A home development workspace provides the setting for this log analysis exercise. The embedded code is a reference implementation based on the student guide and uses fictional data.

## Problem statement

The home development needs a reproducible log analysis workflow for an event modeled on Azure Blob Storage.

## Project objectives

- Count event levels in a synthetic log and identify the most frequent error.
- Export an auditable JSON report without overwriting an existing one.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Azure Blob Storage event format, a simulated local object store, JSON, and Python standard-library modules.

## System architecture

`Synthetic event → object address → local fixture lookup → validated workflow → report`

## Features

- Count event levels in a synthetic log and identify the most frequent error.
- Export an auditable JSON report without overwriting an existing one.

## Prerequisites

Use Python 3.10 or later. The offline practice run requires no cloud account or packages. Use a text editor and a local practice directory.

## Complete Azure source code

```python
"""Local Azure reference implementation for project 015164."""


from collections import Counter
from pathlib import Path
from urllib.parse import unquote, urlsplit
import argparse
import json
import sys


PROJECT_ID = "015164"


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
    """Count event levels in a synthetic log and identify the most frequent error."""
    levels = Counter()
    errors = Counter()
    for line in labels(record, "events"):
        level, delimiter, message = line.partition("|")
        level, message = level.strip(), message.strip()
        if not delimiter or level not in {"INFO", "WARN", "ERROR"} or not message:
            raise ValueError("expected INFO, WARN or ERROR followed by | and a message")
        levels[level] += 1
        if level == "ERROR":
            errors[message] += 1
    return {"counts": dict(sorted(levels.items())),
            "top_error": min(errors, key=lambda key: (-errors[key], key), default=None)}


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
    return {"project": PROJECT_ID, "asset": label(record, "asset"), "result": evaluate(record)}, 0


def main():
    parser = argparse.ArgumentParser(description='Count event levels in a synthetic log and identify the most frequent error.')
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

The adapter reads the Event Grid BlobCreated notification and extracts an object address. It uses that address to read from a local dictionary. The log analysis workflow then evaluates the selected payload.

## Sample input data

Save the synthetic event and object fixture together as `sample.json`.

```json
{
  "event": {
    "eventType": "Microsoft.Storage.BlobCreated",
    "data": {
      "url": "https://portfolioexample.blob.core.windows.net/practice/inputs/portfolio%20015164.json"
    }
  },
  "objects": {
    "https://portfolioexample.blob.core.windows.net/practice/inputs/portfolio 015164.json": {
      "asset": "DEV-05",
      "events": [
        "INFO | Started",
        "ERROR | Timeout",
        "ERROR | Timeout",
        "WARN | Retrying"
      ]
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
  "top_error": "Timeout"
}
```

## Testing and validation

Actual tests ran the Python program against a local simulated object store. The sample field matched the expected value and repeated. The saved JSON matched the terminal output, and an overwrite attempt was rejected.

```json
{
  "top_error": "Timeout"
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

Practice log analysis using Azure and check a repeatable reporting workflow in the home development scenario.

## Conclusion

The project records an observable log analysis result for training case `DEV-05`.

[Back to the Azure project index](../README.md)

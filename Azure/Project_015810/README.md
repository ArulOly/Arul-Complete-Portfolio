# Azure: Documentation Generation — Usability Improvement for Small Business (Project 015810)

## Project overview

Local shop operations provides the setting for this documentation generation exercise. This exercise is based on the student guide and uses fictional data. This project includes source code, sample data, and instructions to run it.

## Problem statement

The small business needs a reproducible documentation generation workflow for an event modeled on Azure Blob Storage.

## Project objectives

- Generate a short Markdown handover from a synthetic operating procedure.
- Print a readable result summary and provide optional JSON output.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Azure Blob Storage event format, a simulated local object store, JSON, and Python standard-library modules.

## System architecture

`Synthetic event → object address → local fixture lookup → validated workflow → report`

## Features

- Generate a short Markdown handover from a synthetic operating procedure.
- Print a readable result summary and provide optional JSON output.

## Prerequisites

Use Python 3.10 or later. The offline practice run requires no cloud account or packages. Use a text editor and a local practice directory.

## Complete Azure source code

```python
"""Local Azure reference implementation for project 015810."""


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
    parser = argparse.ArgumentParser(description='Generate a short Markdown handover from a synthetic operating procedure.')
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

The adapter reads the Event Grid BlobCreated notification and extracts an object address. It uses that address to read from a local dictionary. The documentation generation workflow then evaluates the selected payload.

## Sample input data

Save the synthetic event and object fixture together as `sample.json`.

```json
{
  "event": {
    "eventType": "Microsoft.Storage.BlobCreated",
    "data": {
      "url": "https://portfolioexample.blob.core.windows.net/practice/inputs/portfolio%20015810.json"
    }
  },
  "objects": {
    "https://portfolioexample.blob.core.windows.net/practice/inputs/portfolio 015810.json": {
      "asset": "SHOP-22",
      "summary": "Review the training asset SHOP-22 before handover.",
      "steps": [
        "Confirm the asset label",
        "Record the handover status"
      ]
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
  "steps": 2
}
```

## Testing and validation

Actual tests ran the Python program against a local simulated object store. The sample field matched the expected value and repeated. An invalid record was rejected, and the default terminal view labeled the computed field.

```json
{
  "steps": 2
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

Practice documentation generation using Azure and check a repeatable usability improvement workflow in the small business scenario.

## Conclusion

The project records an observable documentation generation result for training case `SHOP-22`.

[Back to the Azure project index](../README.md)

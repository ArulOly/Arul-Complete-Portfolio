# Azure: Data Validation — Usability Improvement for School Lab (Project 015755)

## Project overview

Shared classroom equipment provides the setting for this data validation exercise. The embedded code is a reference implementation based on the student guide and uses fictional data.

## Problem statement

The school lab needs a reproducible data validation workflow for an event modeled on Azure Blob Storage.

## Project objectives

- Return accepted values and row-level diagnostics for invalid or duplicate codes.
- Print a readable result summary and provide optional JSON output.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Azure Blob Storage event format, a simulated local object store, JSON, and Python standard-library modules.

## System architecture

`Synthetic event → object address → local fixture lookup → validated workflow → report`

## Features

- Return accepted values and row-level diagnostics for invalid or duplicate codes.
- Print a readable result summary and provide optional JSON output.

## Prerequisites

Use Python 3.10 or later. The offline practice run requires no cloud account or packages. Use a text editor and a local practice directory.

## Complete Azure source code

```python
"""Local Azure reference implementation for project 015755."""


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
    """Return accepted values and row-level diagnostics for invalid or duplicate codes."""
    accepted = {}
    rejected = []
    for position, row in enumerate(objects(record, "rows"), 1):
        try:
            code = label(row, "code").upper()
            count = integer(row, "count")
            if code in accepted:
                raise ValueError("duplicate code " + code)
            accepted[code] = count
        except ValueError as exc:
            rejected.append({"row": position, "error": str(exc)})
    return {"accepted": accepted, "rejected": rejected}


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
    parser = argparse.ArgumentParser(description='Return accepted values and row-level diagnostics for invalid or duplicate codes.')
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

The adapter reads the Event Grid BlobCreated notification and extracts an object address. It uses that address to read from a local dictionary. The data validation workflow then evaluates the selected payload.

## Sample input data

Save the synthetic event and object fixture together as `sample.json`.

```json
{
  "event": {
    "eventType": "Microsoft.Storage.BlobCreated",
    "data": {
      "url": "https://portfolioexample.blob.core.windows.net/practice/inputs/portfolio%20015755.json"
    }
  },
  "objects": {
    "https://portfolioexample.blob.core.windows.net/practice/inputs/portfolio 015755.json": {
      "asset": "LAB-17",
      "rows": [
        {
          "code": "KIT-01",
          "count": 4
        },
        {
          "code": "KIT-01",
          "count": 2
        },
        {
          "code": "KIT-02",
          "count": -1
        }
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
  "accepted": {
    "KIT-01": 4
  }
}
```

## Testing and validation

Actual tests ran the Python program against a local simulated object store. The sample field matched the expected value and repeated. An invalid record was rejected, and the default terminal view labeled the computed field.

```json
{
  "accepted": {
    "KIT-01": 4
  }
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

Practice data validation using Azure and check a repeatable usability improvement workflow in the school lab scenario.

## Conclusion

The project records an observable data validation result for training case `LAB-17`.

[Back to the Azure project index](../README.md)

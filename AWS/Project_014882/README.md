# AWS: Automation — Automation for Single-Host Lab (Project 014882)

## Project overview

One simulated lab host provides the setting for this automation exercise. This exercise is based on the student guide and uses fictional data. This project includes source code, sample data, and instructions to run it.

## Problem statement

The single-host lab needs a reproducible automation workflow for an event modeled on Amazon S3.

## Project objectives

- Resolve a local task plan and propagate prerequisite failures to dependent work.
- Process a batch and keep successful rows alongside diagnostics.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Amazon S3 event format, a simulated local object store, JSON, and Python standard-library modules.

## System architecture

`Synthetic event → object address → local fixture lookup → validated workflow → report`

## Features

- Resolve a local task plan and propagate prerequisite failures to dependent work.
- Process a batch and keep successful rows alongside diagnostics.

## Prerequisites

Use Python 3.10 or later. The offline practice run requires no cloud account or packages. Use a text editor and a local practice directory.

## Complete AWS source code

```python
"""Local AWS reference implementation for project 014882."""


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


def labels(record, key, allow_empty=False):
    return [label({key: item}, key) for item in array(record, key, allow_empty)]


def evaluate(record):
    """Resolve a local task plan and propagate prerequisite failures to dependent work."""
    pending = {}
    for task in objects(record, "tasks"):
        name = label(task, "name")
        needs = labels(task, "needs", allow_empty=True)
        ready = task.get("ready")
        if type(ready) is not bool or name in pending:
            raise ValueError("each task needs a unique name and Boolean ready flag")
        pending[name] = (needs, ready)
    known = set(pending)
    if any(set(needs) - known for needs, _ in pending.values()):
        raise ValueError("a prerequisite task is missing")
    states = {}
    while pending:
        eligible = [name for name, (needs, _) in pending.items() if set(needs) <= states.keys()]
        if not eligible:
            raise ValueError("task dependencies contain a cycle")
        for name in eligible:
            needs, ready = pending.pop(name)
            if any(states[need] != "done" for need in needs):
                states[name] = "blocked"
            else:
                states[name] = "done" if ready else "failed"
    return {"task_states": states}


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
    parser = argparse.ArgumentParser(description='Resolve a local task plan and propagate prerequisite failures to dependent work.')
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

The adapter reads the S3 event notification and extracts an object address. It uses that address to read from a local dictionary. The automation workflow then evaluates the selected payload.

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
            "key": "inputs/portfolio+014882.json"
          }
        }
      }
    ]
  },
  "objects": {
    "s3://portfolio-example/inputs/portfolio 014882.json": {
      "records": [
        {
          "asset": "HOST-03",
          "tasks": [
            {
              "name": "collect",
              "ready": true,
              "needs": []
            },
            {
              "name": "verify",
              "ready": false,
              "needs": [
                "collect"
              ]
            },
            {
              "name": "publish",
              "ready": true,
              "needs": [
                "verify"
              ]
            }
          ]
        },
        {
          "asset": "HOST-03-B",
          "tasks": [
            {
              "name": "collect",
              "ready": true,
              "needs": []
            },
            {
              "name": "verify",
              "ready": false,
              "needs": [
                "collect"
              ]
            },
            {
              "name": "publish",
              "ready": true,
              "needs": [
                "verify"
              ]
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
  "task_states": {
    "collect": "done",
    "verify": "failed",
    "publish": "blocked"
  }
}
```

## Testing and validation

Actual tests ran the Python program against a local simulated object store. The sample field matched the expected value and repeated. A mixed batch preserved its successful row and returned exit status 1.

```json
{
  "task_states": {
    "collect": "done",
    "verify": "failed",
    "publish": "blocked"
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

Practice automation using AWS and check a repeatable automation workflow in the single-host lab scenario.

## Conclusion

The project records an observable automation result for training case `HOST-03`.

[Back to the AWS project index](../README.md)

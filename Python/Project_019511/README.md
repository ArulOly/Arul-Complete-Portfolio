# Python: Access Review — Baseline Setup for Home Development (Project 019511)

## Project overview

A home development workspace provides the setting for this access review exercise. The code is an AI-generated reference implementation based on the student guide. Its inputs describe fictional operations.

## Problem statement

A home development workspace needs a reproducible way to handle access review from local records. The exercise focuses on baseline setup and uses `DEV-05` as its sample case.

## Project objectives

- Compare synthetic active grants with an approved role policy.
- Establish a repeatable reference run for one input record.
- Verify an expected result, repeat the run, and observe how invalid input is handled.

## Technologies used

Python 3.10 or later, JSON, and the standard-library modules `pathlib`, `argparse`, `json`, `sys`.

## System architecture

`Local JSON → input loading → workflow validation and calculation → baseline setup result → terminal`

## Features

- Compare synthetic active grants with an approved role policy.
- The terminal shows a reference result for a single snapshot.

## Prerequisites

Install Python 3.10 or later and have a text editor and a writable local practice directory. Use synthetic records or data you have permission to use.

## Complete Python source code

```python
"""Local reference demonstration for Python project 019511."""


from pathlib import Path
import argparse
import json
import sys


PROJECT_ID = '019511'
SCENARIO = 'Home Development'


def load(path):
    with path.open(encoding="utf-8") as handle:
        record = json.load(handle)
    if not isinstance(record, dict):
        raise ValueError("expected a JSON object")
    return record


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
    """Compare synthetic active grants with an approved role policy."""
    approved = set(labels(record, "approved", allow_empty=True))
    grants = set(labels(record, "grants", allow_empty=True))
    return {"excess_grants": sorted(grants - approved),
            "missing_grants": sorted(approved - grants),
            "matches_policy": approved == grants}


def main():
    parser = argparse.ArgumentParser(description='Compare synthetic active grants with an approved role policy. Establish a repeatable reference run for one input record.')
    parser.add_argument("input", type=Path, help="path to a UTF-8 JSON input file")
    args = parser.parse_args()
    status = 0
    try:
        record = load(args.input)
        result = {"baseline_for": label(record, "asset"), "result": evaluate(record)}
    except (OSError, ValueError, OverflowError) as exc:
        print(f"Input or output error: {exc}", file=sys.stderr)
        return 2
    print(json.dumps(result, indent=2, ensure_ascii=False))
    return status


if __name__ == "__main__":
    raise SystemExit(main())
```

## Implementation explanation

`evaluate()` applies the access review rules to a single record. Input JSON supplies one record and the CLI identifies the case so later runs can be compared. Numeric thresholds and other workflow settings come from the sample JSON so they can be changed and examined independently of the implementation.

## Sample input data

Save this synthetic input as `sample.json`.

```json
{
  "asset": "DEV-05",
  "approved": [
    "read",
    "submit"
  ],
  "grants": [
    "read",
    "admin"
  ]
}
```

## Execution instructions

1. Save the Python block to `project.py` in your practice directory.
2. Save the sample JSON in the same directory.
3. Run the following command from that directory.

```sh
python project.py sample.json
```

## Example output

Illustrative excerpt for the sample above.

```json
{
  "excess_grants": [
    "admin"
  ]
}
```

The output envelope also identifies the input case or batch row.

## Testing and validation

Actual validation used Python 3.12 in a temporary directory. The extracted program compiled. The sample returned status 0, and the test asserted the `excess_grants` value shown below. A repeat run produced identical standard output. An incomplete record returned status 2 with a diagnostic.

```json
{
  "excess_grants": [
    "admin"
  ]
}
```

## Error handling

Required fields are validated before their calculation. The CLI writes file and input errors to standard error and returns status 2.

## Troubleshooting

- For a missing file, check the command's working directory and the input filename.
- For a rejected record, compare its fields and types with the sample and read the diagnostic.
- For an unexpected result, retry with the original sample before changing one setting at a time.

## Limitations

The project uses local, in-memory training data in the documented format. It represents one workflow and one operating scenario with synthetic values.

## Learning outcomes

Practice reading structured inputs, validating workflow data, applying access review rules, and reproducing a baseline setup run.

## Conclusion

The sample demonstrates access review in the home development scenario and provides an observable result for baseline setup.

[Back to the Python project index](../README.md)

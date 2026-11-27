# Python: Data Export — Baseline Setup for Small Business (Project 019306)

## Project overview

Local shop operations provides the setting for this data export exercise. The code is an AI-generated reference implementation based on the student guide. Its inputs describe fictional operations.

## Problem statement

Local shop operations needs a reproducible way to handle data export from local records. The exercise focuses on baseline setup and uses `SHOP-22` as its sample case.

## Project objectives

- Convert structured records into a predictable two-column CSV.
- Establish a repeatable reference run for one input record.
- Verify an expected result, repeat the run, and observe how invalid input is handled.

## Technologies used

Python 3.10 or later, JSON, and the standard-library modules `pathlib`, `argparse`, `csv`, `io`, `json`, `sys`.

## System architecture

`Local JSON → input loading → workflow validation and calculation → baseline setup result → terminal`

## Features

- Convert structured records into a predictable two-column CSV.
- The terminal shows a reference result for a single snapshot.

## Prerequisites

Install Python 3.10 or later and have a text editor and a writable local practice directory. Use synthetic records or data you have permission to use.

## Complete Python source code

```python
"""Local reference demonstration for Python project 019306."""


from pathlib import Path
import argparse
import csv
import io
import json
import sys


PROJECT_ID = '019306'
SCENARIO = 'Small Business'


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


def main():
    parser = argparse.ArgumentParser(description='Convert structured records into a predictable two-column CSV. Establish a repeatable reference run for one input record.')
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

`evaluate()` applies the data export rules to a single record. Input JSON supplies one record and the CLI identifies the case so later runs can be compared. Numeric thresholds and other workflow settings come from the sample JSON so they can be changed and examined independently of the implementation.

## Sample input data

Save this synthetic input as `sample.json`.

```json
{
  "asset": "SHOP-22",
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
  "csv": "code,quantity\nKIT-01,4\nKIT-02,2\n"
}
```

The output envelope also identifies the input case or batch row.

## Testing and validation

Actual validation used Python 3.12 in a temporary directory. The extracted program compiled. The sample returned status 0, and the test asserted the `csv` value shown below. A repeat run produced identical standard output. An incomplete record returned status 2 with a diagnostic.

```json
{
  "csv": "code,quantity\nKIT-01,4\nKIT-02,2\n"
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

Practice reading structured inputs, validating workflow data, applying data export rules, and reproducing a baseline setup run.

## Conclusion

The sample demonstrates data export in the small business scenario and provides an observable result for baseline setup.

[Back to the Python project index](../README.md)

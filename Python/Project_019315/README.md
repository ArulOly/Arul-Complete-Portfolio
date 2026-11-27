# Python: Data Export — Usability Improvement for Home Development (Project 019315)

## Project overview

A home development workspace provides the setting for this data export exercise. The code is an AI-generated reference implementation based on the student guide. Its inputs describe fictional operations.

## Problem statement

A home development workspace needs a reproducible way to handle data export from local records. The exercise focuses on usability improvement and uses `DEV-05` as its sample case.

## Project objectives

- Convert structured records into a predictable two-column CSV.
- Provide a discoverable help screen and a readable terminal summary.
- Verify an expected result, repeat the run, and observe how invalid input is handled.

## Technologies used

Python 3.10 or later, JSON, and the standard-library modules `pathlib`, `argparse`, `csv`, `io`, `json`, `sys`.

## System architecture

`Local JSON → input loading → workflow validation and calculation → usability improvement result → terminal`

## Features

- Convert structured records into a predictable two-column CSV.
- The operator can use --help to inspect arguments and --json for programmatic review.

## Prerequisites

Install Python 3.10 or later and have a text editor and a writable local practice directory. Use synthetic records or data you have permission to use.

## Complete Python source code

```python
"""Local reference demonstration for Python project 019315."""


from pathlib import Path
import argparse
import csv
import io
import json
import sys


PROJECT_ID = '019315'
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
    parser = argparse.ArgumentParser(description='Convert structured records into a predictable two-column CSV. Provide a discoverable help screen and a readable terminal summary.')
    parser.add_argument("input", type=Path, help="path to a UTF-8 JSON input file")
    parser.add_argument("--json", action="store_true", help="print JSON")
    args = parser.parse_args()
    status = 0
    try:
        record = load(args.input)
        result = {"asset": label(record, "asset"), "result": evaluate(record)}
        if not args.json:
            print(f"{SCENARIO} | {result['asset']}")
            for key, value in result["result"].items():
                display = json.dumps(value, ensure_ascii=False)
                print(f"{key.replace('_', ' ').title()}: {display}")
            return status
    except (OSError, ValueError, OverflowError) as exc:
        print(f"Input or output error: {exc}", file=sys.stderr)
        return 2
    print(json.dumps(result, indent=2, ensure_ascii=False))
    return status


if __name__ == "__main__":
    raise SystemExit(main())
```

## Implementation explanation

`evaluate()` applies the data export rules to a single record. The CLI converts each result field into a labeled line or outputs JSON with a flag. Numeric thresholds and other workflow settings come from the sample JSON so they can be changed and examined independently of the implementation.

## Sample input data

Save this synthetic input as `sample.json`.

```json
{
  "asset": "DEV-05",
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

Omit `--json` for a labeled terminal view. Run `python project.py --help` for argument help.

```sh
python project.py sample.json --json
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

Actual validation used Python 3.12 in a temporary directory. The extracted program compiled. The sample returned status 0, and the test asserted the `csv` value shown below. A repeat run produced identical standard output. An incomplete record returned status 2 with a diagnostic. Help displayed the JSON flag and the default view labeled its result fields.

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

Practice reading structured inputs, validating workflow data, applying data export rules, and reproducing a usability improvement run.

## Conclusion

The sample demonstrates data export in the home development scenario and provides an observable result for usability improvement.

[Back to the Python project index](../README.md)

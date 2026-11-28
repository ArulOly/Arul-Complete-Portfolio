# Python: Audit — Usability Improvement for Home Development (Project 019115)

## Project overview

A home development workspace provides the setting for this audit exercise. This exercise is based on the student guide and uses fictional data. This project includes source code, sample data, and instructions to run it.

## Problem statement

A home development workspace needs a reproducible way to handle audit from local records. The exercise focuses on usability improvement and uses `DEV-05` as its sample case.

## Project objectives

- Check whether recorded asset reviews meet a cutoff and list departures.
- Provide a discoverable help screen and a readable terminal summary.
- Verify an expected result, repeat the run, and observe how invalid input is handled.

## Technologies used

Python 3.10 or later, JSON, and the standard-library modules `datetime`, `pathlib`, `argparse`, `json`, `sys`.

## System architecture

`Local JSON → input loading → workflow validation and calculation → usability improvement result → terminal`

## Features

- Check whether recorded asset reviews meet a cutoff and list departures.
- The operator can use --help to inspect arguments and --json for programmatic review.

## Prerequisites

Install Python 3.10 or later and have a text editor and a writable local practice directory. Use synthetic records or data you have permission to use.

## Complete Python source code

```python
"""Local reference demonstration for Python project 019115."""


from datetime import date
from pathlib import Path
import argparse
import json
import sys


PROJECT_ID = '019115'
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


def iso_date(record, key):
    value = label(record, key)
    try:
        parsed = date.fromisoformat(value)
    except ValueError as exc:
        raise ValueError(f"{key} must use YYYY-MM-DD") from exc
    if parsed.isoformat() != value:
        raise ValueError(f"{key} must use YYYY-MM-DD")
    return parsed


def evaluate(record):
    """Check whether recorded asset reviews meet a cutoff and list departures."""
    reviewed = iso_date(record, "reviewed_on")
    cutoff = iso_date(record, "cutoff")
    return {"owner": label(record, "owner"), "reviewed_on": reviewed.isoformat(),
            "follow_up": reviewed < cutoff}


def main():
    parser = argparse.ArgumentParser(description='Check whether recorded asset reviews meet a cutoff and list departures. Provide a discoverable help screen and a readable terminal summary.')
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

`evaluate()` applies the audit rules to a single record. The CLI converts each result field into a labeled line or outputs JSON with a flag. Numeric thresholds and other workflow settings come from the sample JSON so they can be changed and examined independently of the implementation.

## Sample input data

Save this synthetic input as `sample.json`.

```json
{
  "asset": "DEV-05",
  "reviewed_on": "2026-05-03",
  "cutoff": "2026-06-01",
  "owner": "Lab Operations"
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
  "follow_up": true
}
```

The output envelope also identifies the input case or batch row.

## Testing and validation

Actual validation used Python 3.12 in a temporary directory. The extracted program compiled. The sample returned status 0, and the test asserted the `follow_up` value shown below. A repeat run produced identical standard output. An incomplete record returned status 2 with a diagnostic. Help displayed the JSON flag and the default view labeled its result fields.

```json
{
  "follow_up": true
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

Practice reading structured inputs, validating workflow data, applying audit rules, and reproducing a usability improvement run.

## Conclusion

The sample demonstrates audit in the home development scenario and provides an observable result for usability improvement.

[Back to the Python project index](../README.md)

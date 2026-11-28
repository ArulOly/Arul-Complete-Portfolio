# Python: Scheduled Reporting — Recovery for Help Desk (Project 019493)

## Project overview

A help desk training queue provides the setting for this scheduled reporting exercise. This exercise is based on the student guide and uses fictional data. This project includes source code, sample data, and instructions to run it.

## Problem statement

A help desk training queue needs a reproducible way to handle scheduled reporting from local records. The exercise focuses on recovery and uses `DESK-14` as its sample case.

## Project objectives

- Calculate the next due date from an explicit reporting interval and reference date.
- Try a fallback input snapshot when the primary input is missing or invalid.
- Verify an expected result, repeat the run, and observe how invalid input is handled.

## Technologies used

Python 3.10 or later, JSON, and the standard-library modules `datetime`, `pathlib`, `argparse`, `json`, `sys`.

## System architecture

`Local JSON → input loading → workflow validation and calculation → recovery result → terminal`

## Features

- Calculate the next due date from an explicit reporting interval and reference date.
- The output identifies the input role and gives the reason when the fallback is used.

## Prerequisites

Install Python 3.10 or later and have a text editor and a writable local practice directory. Use synthetic records or data you have permission to use.

## Complete Python source code

```python
"""Local reference demonstration for Python project 019493."""


from datetime import date
from datetime import timedelta
from pathlib import Path
import argparse
import json
import sys


PROJECT_ID = '019493'
SCENARIO = 'Help Desk'


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
    """Calculate the next due date from an explicit reporting interval and reference date."""
    previous = iso_date(record, "previous_report")
    as_of = iso_date(record, "as_of")
    interval = integer(record, "interval_days", minimum=1, maximum=365)
    if as_of < previous:
        raise ValueError("as_of must be on or after previous_report")
    due = previous + timedelta(days=interval)
    return {"due_on": due.isoformat(), "due": as_of >= due,
            "days_overdue": max(0, (as_of - due).days)}


def main():
    parser = argparse.ArgumentParser(description='Calculate the next due date from an explicit reporting interval and reference date. Try a fallback input snapshot when the primary input is missing or invalid.')
    parser.add_argument("input", type=Path, help="path to a UTF-8 JSON input file")
    parser.add_argument("--fallback", type=Path, required=True, help="valid fallback JSON")
    args = parser.parse_args()
    status = 0
    try:
        try:
            record = load(args.input)
            result = {"source": "primary", "asset": label(record, "asset"),
                      "result": evaluate(record)}
        except (OSError, ValueError, OverflowError) as exc:
            record = load(args.fallback)
            result = {"source": "fallback", "asset": label(record, "asset"),
                      "primary_error": str(exc), "result": evaluate(record)}
    except (OSError, ValueError, OverflowError) as exc:
        print(f"Input or output error: {exc}", file=sys.stderr)
        return 2
    print(json.dumps(result, indent=2, ensure_ascii=False))
    return status


if __name__ == "__main__":
    raise SystemExit(main())
```

## Implementation explanation

`evaluate()` applies the scheduled reporting rules to a single record. The primary file is evaluated first. An input error triggers a fresh read of the fallback. Numeric thresholds and other workflow settings come from the sample JSON so they can be changed and examined independently of the implementation.

## Sample input data

Save this synthetic input as `sample.json`.

```json
{
  "asset": "DESK-14",
  "previous_report": "2026-06-01",
  "as_of": "2026-06-16",
  "interval_days": 7
}
```

## Execution instructions

1. Save the Python block to `project.py` in your practice directory.
2. Save the sample JSON in the same directory.
3. Run the following command from that directory.

Save a second copy of the sample as `fallback.json`. To exercise recovery, point the first argument at a missing file.

```sh
python project.py sample.json --fallback fallback.json
```

## Example output

Illustrative excerpt for the sample above.

```json
{
  "days_overdue": 8
}
```

The output envelope also identifies the input case or batch row.

## Testing and validation

Actual validation used Python 3.12 in a temporary directory. The extracted program compiled. The sample returned status 0, and the test asserted the `days_overdue` value shown below. A repeat run produced identical standard output. An incomplete primary record used the fallback. An unavailable fallback returned status 2.

```json
{
  "days_overdue": 8
}
```

## Error handling

Required fields are validated before their calculation. A missing or invalid primary file triggers fallback evaluation. Failure of that input returns status 2.

## Troubleshooting

- For a missing file, check the command's working directory and the input filename.
- For a rejected record, compare its fields and types with the sample and read the diagnostic.
- For an unexpected result, retry with the original sample before changing one setting at a time.

## Limitations

The project uses local, in-memory training data in the documented format. It represents one workflow and one operating scenario with synthetic values.

## Learning outcomes

Practice reading structured inputs, validating workflow data, applying scheduled reporting rules, and reproducing a recovery run.

## Conclusion

The sample demonstrates scheduled reporting in the help desk scenario and provides an observable result for recovery.

[Back to the Python project index](../README.md)

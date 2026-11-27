# Python: Visual Reporting — Recovery for School Lab (Project 019903)

## Project overview

Shared classroom equipment provides the setting for this visual reporting exercise. The code is an AI-generated reference implementation based on the student guide. Its inputs describe fictional operations.

## Problem statement

Shared classroom equipment needs a reproducible way to handle visual reporting from local records. The exercise focuses on recovery and uses `LAB-17` as its sample case.

## Project objectives

- Create a proportional text bar chart with exact values and a labeled scale.
- Try a fallback input snapshot when the primary input is missing or invalid.
- Verify an expected result, repeat the run, and observe how invalid input is handled.

## Technologies used

Python 3.10 or later, JSON, and the standard-library modules `pathlib`, `argparse`, `json`, `sys`.

## System architecture

`Local JSON → input loading → workflow validation and calculation → recovery result → terminal`

## Features

- Create a proportional text bar chart with exact values and a labeled scale.
- The output identifies the input role and gives the reason when the fallback is used.

## Prerequisites

Install Python 3.10 or later and have a text editor and a writable local practice directory. Use synthetic records or data you have permission to use.

## Complete Python source code

```python
"""Local reference demonstration for Python project 019903."""


from pathlib import Path
import argparse
import json
import sys


PROJECT_ID = '019903'
SCENARIO = 'School Lab'


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


def mapping(record, key):
    value = record.get(key)
    if not isinstance(value, dict):
        raise ValueError(f"{key} must be an object")
    return value


def evaluate(record):
    """Create a proportional text bar chart with exact values and a labeled scale."""
    values = mapping(record, "values")
    if not values:
        raise ValueError("values must contain at least one category")
    counts = {key: integer(values, key) for key in values}
    width = integer(record, "width", minimum=1, maximum=60)
    peak = max(counts.values())
    bars = [f"{key}: {'#' * ((value * width + peak // 2) // peak if peak else 0)} ({value})"
            for key, value in sorted(counts.items())]
    return {"peak": peak, "max_bar_width": width, "bars": bars}


def main():
    parser = argparse.ArgumentParser(description='Create a proportional text bar chart with exact values and a labeled scale. Try a fallback input snapshot when the primary input is missing or invalid.')
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

`evaluate()` applies the visual reporting rules to a single record. The primary file is evaluated first. An input error triggers a fresh read of the fallback. Numeric thresholds and other workflow settings come from the sample JSON so they can be changed and examined independently of the implementation.

## Sample input data

Save this synthetic input as `sample.json`.

```json
{
  "asset": "LAB-17",
  "values": {
    "available": 12,
    "reserved": 7,
    "repair": 3
  },
  "width": 12
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
  "peak": 12
}
```

The output envelope also identifies the input case or batch row.

## Testing and validation

Actual validation used Python 3.12 in a temporary directory. The extracted program compiled. The sample returned status 0, and the test asserted the `peak` value shown below. A repeat run produced identical standard output. An incomplete primary record used the fallback. An unavailable fallback returned status 2.

```json
{
  "peak": 12
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

Practice reading structured inputs, validating workflow data, applying visual reporting rules, and reproducing a recovery run.

## Conclusion

The sample demonstrates visual reporting in the school lab scenario and provides an observable result for recovery.

[Back to the Python project index](../README.md)

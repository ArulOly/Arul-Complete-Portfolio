# Python: Alerting — Recovery for Portfolio Demo (Project 019448)

## Project overview

A repeatable portfolio demonstration provides the setting for this alerting exercise. This exercise is based on the student guide and uses fictional data. This project includes source code, sample data, and instructions to run it.

## Problem statement

A repeatable portfolio demonstration needs a reproducible way to handle alerting from local records. The exercise focuses on recovery and uses `DEMO-02` as its sample case.

## Project objectives

- Emit one local alert per consecutive breach episode after a configured threshold.
- Try a fallback input snapshot when the primary input is missing or invalid.
- Verify an expected result, repeat the run, and observe how invalid input is handled.

## Technologies used

Python 3.10 or later, JSON, and the standard-library modules `pathlib`, `argparse`, `json`, `sys`.

## System architecture

`Local JSON → input loading → workflow validation and calculation → recovery result → terminal`

## Features

- Emit one local alert per consecutive breach episode after a configured threshold.
- The output identifies the input role and gives the reason when the fallback is used.

## Prerequisites

Install Python 3.10 or later and have a text editor and a writable local practice directory. Use synthetic records or data you have permission to use.

## Complete Python source code

```python
"""Local reference demonstration for Python project 019448."""


from pathlib import Path
import argparse
import json
import sys


PROJECT_ID = '019448'
SCENARIO = 'Portfolio Demo'


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


def series(record, key, maximum=1_000_000_000):
    return [integer({key: item}, key, maximum=maximum) for item in array(record, key)]


def evaluate(record):
    """Emit one local alert per consecutive breach episode after a configured threshold."""
    limit = integer(record, "limit")
    needed = integer(record, "consecutive", minimum=1)
    streak = 0
    alerts = []
    for index, value in enumerate(series(record, "readings"), 1):
        streak = streak + 1 if value > limit else 0
        if streak == needed:
            alerts.append({"sample": index, "reading": value})
    return {"limit": limit, "alerts": alerts}


def main():
    parser = argparse.ArgumentParser(description='Emit one local alert per consecutive breach episode after a configured threshold. Try a fallback input snapshot when the primary input is missing or invalid.')
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

`evaluate()` applies the alerting rules to a single record. The primary file is evaluated first. An input error triggers a fresh read of the fallback. Numeric thresholds and other workflow settings come from the sample JSON so they can be changed and examined independently of the implementation.

## Sample input data

Save this synthetic input as `sample.json`.

```json
{
  "asset": "DEMO-02",
  "readings": [
    10,
    22,
    24,
    25,
    8,
    26,
    29
  ],
  "limit": 20,
  "consecutive": 2
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
  "alerts": [
    {
      "sample": 3,
      "reading": 24
    },
    {
      "sample": 7,
      "reading": 29
    }
  ]
}
```

The output envelope also identifies the input case or batch row.

## Testing and validation

Actual validation used Python 3.12 in a temporary directory. The extracted program compiled. The sample returned status 0, and the test asserted the `alerts` value shown below. A repeat run produced identical standard output. An incomplete primary record used the fallback. An unavailable fallback returned status 2.

```json
{
  "alerts": [
    {
      "sample": 3,
      "reading": 24
    },
    {
      "sample": 7,
      "reading": 29
    }
  ]
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

Practice reading structured inputs, validating workflow data, applying alerting rules, and reproducing a recovery run.

## Conclusion

The sample demonstrates alerting in the portfolio demo scenario and provides an observable result for recovery.

[Back to the Python project index](../README.md)

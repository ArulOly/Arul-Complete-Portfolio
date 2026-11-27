# Python: Service Check — Automation for Offline Classroom (Project 019627)

## Project overview

A classroom operating offline provides the setting for this service check exercise. The code is an AI-generated reference implementation based on the student guide. Its inputs describe fictional operations.

## Problem statement

A classroom operating offline needs a reproducible way to handle service check from local records. The exercise focuses on automation and uses `CLASS-09` as its sample case.

## Project objectives

- Classify synthetic service probes by status and latency.
- Process a batch in order and keep valid results when individual records fail.
- Verify an expected result, repeat the run, and observe how invalid input is handled.

## Technologies used

Python 3.10 or later, JSON, and the standard-library modules `pathlib`, `argparse`, `json`, `sys`.

## System architecture

`Local JSON → input loading → workflow validation and calculation → automation result → terminal`

## Features

- Classify synthetic service probes by status and latency.
- The CLI processes multiple rows in one call and returns status 1 if any row fails.

## Prerequisites

Install Python 3.10 or later and have a text editor and a writable local practice directory. Use synthetic records or data you have permission to use.

## Complete Python source code

```python
"""Local reference demonstration for Python project 019627."""


from pathlib import Path
import argparse
import json
import sys


PROJECT_ID = '019627'
SCENARIO = 'Offline Classroom'


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
    """Classify synthetic service probes by status and latency."""
    budget = integer(record, "max_ms", minimum=1)
    results = []
    for probe in objects(record, "probes"):
        status = integer(probe, "status", minimum=100, maximum=599)
        latency = integer(probe, "ms")
        results.append({"service": label(probe, "service"),
                        "healthy": 200 <= status < 300 and latency <= budget})
    return {"checks": results, "healthy": sum(result["healthy"] for result in results)}


def main():
    parser = argparse.ArgumentParser(description='Classify synthetic service probes by status and latency. Process a batch in order and keep valid results when individual records fail.')
    parser.add_argument("input", type=Path, help="path to a UTF-8 JSON input file")
    args = parser.parse_args()
    status = 0
    try:
        records = array(load(args.input), "records")
        rows = []
        for index, record in enumerate(records, 1):
            try:
                if not isinstance(record, dict):
                    raise ValueError("expected a record object")
                rows.append({"row": index, "asset": label(record, "asset"),
                             "result": evaluate(record)})
            except ValueError as exc:
                rows.append({"row": index, "error": str(exc)})
        result = {"rows": rows}
        status = int(any("error" in row for row in rows))
    except (OSError, ValueError, OverflowError) as exc:
        print(f"Input or output error: {exc}", file=sys.stderr)
        return 2
    print(json.dumps(result, indent=2, ensure_ascii=False))
    return status


if __name__ == "__main__":
    raise SystemExit(main())
```

## Implementation explanation

`evaluate()` applies the service check rules to a single record. The outer records array keeps each result or error associated with its original row number. Numeric thresholds and other workflow settings come from the sample JSON so they can be changed and examined independently of the implementation.

## Sample input data

Save this synthetic input as `sample.json`.

```json
{
  "records": [
    {
      "asset": "CLASS-09",
      "probes": [
        {
          "service": "catalog",
          "status": 200,
          "ms": 25
        },
        {
          "service": "print-queue",
          "status": 503,
          "ms": 60
        }
      ],
      "max_ms": 50
    },
    {
      "asset": "CLASS-09-B",
      "probes": [
        {
          "service": "catalog",
          "status": 200,
          "ms": 25
        },
        {
          "service": "print-queue",
          "status": 503,
          "ms": 60
        }
      ],
      "max_ms": 50
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
  "healthy": 1
}
```

The output envelope also identifies the input case or batch row.

## Testing and validation

Actual validation used Python 3.12 in a temporary directory. The extracted program compiled. The sample returned status 0, and the test asserted the `healthy` value shown below. A repeat run produced identical standard output. A mixed batch preserved its valid result, reported the bad row, and returned status 1.

```json
{
  "healthy": 1
}
```

## Error handling

Required fields are validated before their calculation. A bad batch row produces a row-level error while the remaining rows are evaluated.

## Troubleshooting

- For a missing file, check the command's working directory and the input filename.
- For a rejected record, compare its fields and types with the sample and read the diagnostic.
- For an unexpected result, retry with the original sample before changing one setting at a time.

## Limitations

The project uses local, in-memory training data in the documented format. It represents one workflow and one operating scenario with synthetic values.

## Learning outcomes

Practice reading structured inputs, validating workflow data, applying service check rules, and reproducing a automation run.

## Conclusion

The sample demonstrates service check in the offline classroom scenario and provides an observable result for automation.

[Back to the Python project index](../README.md)

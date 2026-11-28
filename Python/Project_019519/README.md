# Python: Access Review — Reporting for Public Dataset (Project 019519)

## Project overview

A synthetic public-data exercise provides the setting for this access review exercise. The code is a reference implementation based on the student guide. Its inputs describe fictional operations.

## Problem statement

A synthetic public-data exercise needs a reproducible way to handle access review from local records. The exercise focuses on reporting and uses `DATA-12` as its sample case.

## Project objectives

- Compare synthetic active grants with an approved role policy.
- Create a portable JSON report without replacing an existing file.
- Verify an expected result, repeat the run, and observe how invalid input is handled.

## Technologies used

Python 3.10 or later, JSON, and the standard-library modules `pathlib`, `argparse`, `json`, `sys`.

## System architecture

`Local JSON → input loading → workflow validation and calculation → reporting result → terminal`

## Features

- Compare synthetic active grants with an approved role policy.
- The terminal and optional UTF-8 report file carry the same structured result.

## Prerequisites

Install Python 3.10 or later and have a text editor and a writable local practice directory. Use synthetic records or data you have permission to use.

## Complete Python source code

```python
"""Local reference demonstration for Python project 019519."""


from pathlib import Path
import argparse
import json
import sys


PROJECT_ID = '019519'
SCENARIO = 'Public Dataset'


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
    parser = argparse.ArgumentParser(description='Compare synthetic active grants with an approved role policy. Create a portable JSON report without replacing an existing file.')
    parser.add_argument("input", type=Path, help="path to a UTF-8 JSON input file")
    parser.add_argument("--output", type=Path, help="new JSON report path")
    args = parser.parse_args()
    status = 0
    try:
        record = load(args.input)
        result = {"project": PROJECT_ID, "asset": label(record, "asset"),
                  "result": evaluate(record)}
        if args.output is not None:
            with args.output.open("x", encoding="utf-8") as handle:
                json.dump(result, handle, indent=2, ensure_ascii=False)
                handle.write("\n")
    except (OSError, ValueError, OverflowError) as exc:
        print(f"Input or output error: {exc}", file=sys.stderr)
        return 2
    print(json.dumps(result, indent=2, ensure_ascii=False))
    return status


if __name__ == "__main__":
    raise SystemExit(main())
```

## Implementation explanation

`evaluate()` applies the access review rules to a single record. The workflow builds its report before opening the destination in exclusive creation mode. Numeric thresholds and other workflow settings come from the sample JSON so they can be changed and examined independently of the implementation.

## Sample input data

Save this synthetic input as `sample.json`.

```json
{
  "asset": "DATA-12",
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

Choose a fresh output name for each run that writes a report. Omit `--output` to print it only.

```sh
python project.py sample.json --output report.json
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

Actual validation used Python 3.12 in a temporary directory. The extracted program compiled. The sample returned status 0, and the test asserted the `excess_grants` value shown below. A repeat run produced identical standard output. An incomplete record returned status 2 with a diagnostic. The report matched standard output and a second write to the same path was rejected.

```json
{
  "excess_grants": [
    "admin"
  ]
}
```

## Error handling

Required fields are validated before their calculation. The output path is opened in exclusive mode, so existing reports are protected.

## Troubleshooting

- For a missing file, check the command's working directory and the input filename.
- For a rejected record, compare its fields and types with the sample and read the diagnostic.
- For an unexpected result, retry with the original sample before changing one setting at a time.

## Limitations

The project uses local, in-memory training data in the documented format. It represents one workflow and one operating scenario with synthetic values.

## Learning outcomes

Practice reading structured inputs, validating workflow data, applying access review rules, and reproducing a reporting run.

## Conclusion

The sample demonstrates access review in the public dataset scenario and provides an observable result for reporting.

[Back to the Python project index](../README.md)

# Python: Monitoring — Usability Improvement for Portfolio Demo (Project 019100)

## Project overview

A repeatable portfolio demonstration provides the setting for this monitoring exercise. The code is an AI-generated reference implementation based on the student guide. Its inputs describe fictional operations.

## Problem statement

A repeatable portfolio demonstration needs a reproducible way to handle monitoring from local records. The exercise focuses on usability improvement and uses `DEMO-02` as its sample case.

## Project objectives

- Summarize utilization readings and count readings above the limit.
- Provide a discoverable help screen and a readable terminal summary.
- Verify an expected result, repeat the run, and observe how invalid input is handled.

## Technologies used

Python 3.10 or later, JSON, and the standard-library modules `pathlib`, `argparse`, `json`, `statistics`, `sys`.

## System architecture

`Local JSON → input loading → workflow validation and calculation → usability improvement result → terminal`

## Features

- Summarize utilization readings and count readings above the limit.
- The operator can use --help to inspect arguments and --json for programmatic review.

## Prerequisites

Install Python 3.10 or later and have a text editor and a writable local practice directory. Use synthetic records or data you have permission to use.

## Complete Python source code

```python
"""Local reference demonstration for Python project 019100."""


from pathlib import Path
import argparse
import json
import statistics
import sys


PROJECT_ID = '019100'
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
    """Summarize utilization readings and count readings above the limit."""
    readings = series(record, "readings_pct", maximum=100)
    limit = integer(record, "limit_pct", maximum=100)
    return {"mean_pct": round(statistics.mean(readings), 2),
            "peak_pct": max(readings), "above_limit": sum(n > limit for n in readings)}


def main():
    parser = argparse.ArgumentParser(description='Summarize utilization readings and count readings above the limit. Provide a discoverable help screen and a readable terminal summary.')
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

`evaluate()` applies the monitoring rules to a single record. The CLI converts each result field into a labeled line or outputs JSON with a flag. Numeric thresholds and other workflow settings come from the sample JSON so they can be changed and examined independently of the implementation.

## Sample input data

Save this synthetic input as `sample.json`.

```json
{
  "asset": "DEMO-02",
  "readings_pct": [
    35,
    81,
    54
  ],
  "limit_pct": 80
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
  "above_limit": 1
}
```

The output envelope also identifies the input case or batch row.

## Testing and validation

Actual validation used Python 3.12 in a temporary directory. The extracted program compiled. The sample returned status 0, and the test asserted the `above_limit` value shown below. A repeat run produced identical standard output. An incomplete record returned status 2 with a diagnostic. Help displayed the JSON flag and the default view labeled its result fields.

```json
{
  "above_limit": 1
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

Practice reading structured inputs, validating workflow data, applying monitoring rules, and reproducing a usability improvement run.

## Conclusion

The sample demonstrates monitoring in the portfolio demo scenario and provides an observable result for usability improvement.

[Back to the Python project index](../README.md)

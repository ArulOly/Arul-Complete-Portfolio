# Python: Log Analysis — Recovery for Two-Node Lab (Project 019188)

## Project overview

A two-node lab simulation provides the setting for this log analysis exercise. The code is a reference implementation based on the student guide. Its inputs describe fictional operations.

## Problem statement

A two-node lab simulation needs a reproducible way to handle log analysis from local records. The exercise focuses on recovery and uses `NODE-B` as its sample case.

## Project objectives

- Count event levels in a synthetic log and identify the most frequent error.
- Try a fallback input snapshot when the primary input is missing or invalid.
- Verify an expected result, repeat the run, and observe how invalid input is handled.

## Technologies used

Python 3.10 or later, JSON, and the standard-library modules `collections`, `pathlib`, `argparse`, `json`, `sys`.

## System architecture

`Local JSON → input loading → workflow validation and calculation → recovery result → terminal`

## Features

- Count event levels in a synthetic log and identify the most frequent error.
- The output identifies the input role and gives the reason when the fallback is used.

## Prerequisites

Install Python 3.10 or later and have a text editor and a writable local practice directory. Use synthetic records or data you have permission to use.

## Complete Python source code

```python
"""Local reference demonstration for Python project 019188."""


from collections import Counter
from pathlib import Path
import argparse
import json
import sys


PROJECT_ID = '019188'
SCENARIO = 'Two-Node Lab'


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
    """Count event levels in a synthetic log and identify the most frequent error."""
    levels = Counter()
    errors = Counter()
    for line in labels(record, "events"):
        level, delimiter, message = line.partition("|")
        level, message = level.strip(), message.strip()
        if not delimiter or level not in {"INFO", "WARN", "ERROR"} or not message:
            raise ValueError("expected INFO, WARN or ERROR followed by | and a message")
        levels[level] += 1
        if level == "ERROR":
            errors[message] += 1
    return {"counts": dict(sorted(levels.items())),
            "top_error": min(errors, key=lambda key: (-errors[key], key), default=None)}


def main():
    parser = argparse.ArgumentParser(description='Count event levels in a synthetic log and identify the most frequent error. Try a fallback input snapshot when the primary input is missing or invalid.')
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

`evaluate()` applies the log analysis rules to a single record. The primary file is evaluated first. An input error triggers a fresh read of the fallback. Numeric thresholds and other workflow settings come from the sample JSON so they can be changed and examined independently of the implementation.

## Sample input data

Save this synthetic input as `sample.json`.

```json
{
  "asset": "NODE-B",
  "events": [
    "INFO | Started",
    "ERROR | Timeout",
    "ERROR | Timeout",
    "WARN | Retrying"
  ]
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
  "top_error": "Timeout"
}
```

The output envelope also identifies the input case or batch row.

## Testing and validation

Actual validation used Python 3.12 in a temporary directory. The extracted program compiled. The sample returned status 0, and the test asserted the `top_error` value shown below. A repeat run produced identical standard output. An incomplete primary record used the fallback. An unavailable fallback returned status 2.

```json
{
  "top_error": "Timeout"
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

Practice reading structured inputs, validating workflow data, applying log analysis rules, and reproducing a recovery run.

## Conclusion

The sample demonstrates log analysis in the two-node lab scenario and provides an observable result for recovery.

[Back to the Python project index](../README.md)

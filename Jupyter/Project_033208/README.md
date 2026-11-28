# Jupyter: Backup Verification — Recovery for Small Business (Project 033208)

## Project overview

Local shop operations provides the setting for this backup verification exercise. The embedded code is a reference implementation based on the student guide and uses fictional data.

## Problem statement

The small business needs an inspectable notebook for backup verification and recovery.

## Project objectives

- Compare a synthetic backup payload against an expected SHA-256 digest.
- Display the fallback result and record why the primary failed.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Jupyter, IPython rich display, Python standard library, and JSON.

## System architecture

`Local JSON → Python notebook cell → workflow validation → rich notebook display`

## Features

- Compare a synthetic backup payload against an expected SHA-256 digest.
- Display the fallback result and record why the primary failed.

## Prerequisites

Install Python 3.10 or later and JupyterLab or Jupyter Notebook with a Python kernel. Use a text editor and a local practice directory.

## Complete Jupyter source code

```python
# Jupyter reference implementation for project 033208.


from IPython.display import JSON, display
from pathlib import Path
import hashlib
import json


def label(record, key):
    value = record.get(key)
    if not isinstance(value, str) or not value.strip():
        raise ValueError(f"{key} must be non-empty text")
    return value.strip()


def evaluate(record):
    """Compare a synthetic backup payload against an expected SHA-256 digest."""
    expected = label(record, "sha256").lower()
    if len(expected) != 64 or any(char not in "0123456789abcdef" for char in expected):
        raise ValueError("sha256 must contain exactly 64 hexadecimal characters")
    actual = hashlib.sha256(label(record, "payload").encode("utf-8")).hexdigest()
    return {"bytes": len(label(record, "payload").encode("utf-8")),
            "sha256": actual, "verified": actual == expected}


def as_record(value):
    if not isinstance(value, dict):
        raise ValueError("Expected an object")
    return value


def analyze(payload):
    payload = as_record(payload)
    try:
        return {"source": "primary", "result": evaluate(as_record(payload.get("primary")))}
    except (ValueError, OverflowError) as exc:
        return {"source": "fallback", "primary_error": str(exc),
                "result": evaluate(as_record(payload.get("fallback")))}


def run_cell(input_path):
    try:
        payload = json.loads(Path(input_path).read_text(encoding="utf-8"))
        json.dumps(payload, allow_nan=False)
        report = analyze(payload)
        display(JSON(report))
        return report
    except (OSError, ValueError, OverflowError, RecursionError) as exc:
        print(f"Notebook input or output error: {exc}")
        return None


result = run_cell(Path("sample.json"))
```

## Implementation explanation

`evaluate()` calculates the backup verification outcome. `analyze()` applies the recovery rules, and IPython renders the response inside the notebook.

## Sample input data

Save this synthetic input as `sample.json`.

```json
{
  "primary": {
    "asset": "SHOP-22",
    "payload": "Backup manifest for SHOP-22, revision 1",
    "sha256": "89b1aa104bcff1c49c73ac1b9038613f2a2a26cacb23cafad20b419621ba8cde"
  },
  "fallback": {
    "asset": "SHOP-22",
    "payload": "Backup manifest for SHOP-22, revision 1",
    "sha256": "89b1aa104bcff1c49c73ac1b9038613f2a2a26cacb23cafad20b419621ba8cde"
  }
}
```

## Execution instructions

Save the synthetic input as `sample.json` in the notebook folder. Open a new Python notebook in JupyterLab or Jupyter Notebook, paste the source block into one code cell, and run it with **Shift+Enter**. Restart the kernel before a clean repeat.

## Example output

Illustrative excerpt for the synthetic sample.

```json
{
  "verified": true
}
```

## Testing and validation

Actual validation ran the embedded cell in a Jupyter Python kernel with temporary input files. The rendered result matched the expected field and repeat analysis produced the same value. A failed primary used its fallback. Two bad snapshots produced a validation error.

```json
{
  "verified": true
}
```

## Error handling

Input and output errors produce a diagnostic inside the cell. Batch and recovery modes retain row or primary diagnostics in the result.

## Troubleshooting

- Confirm the documented runtime and copy the complete source block.
- Start with the documented sample and use the filename or request path shown above.
- Compare the observed value with the example before changing an input.

## Limitations

The notebook processes small local synthetic fixtures and runs interactively.

## Learning outcomes

Practice backup verification using Jupyter and check a repeatable recovery workflow in the small business scenario.

## Conclusion

The project records an observable backup verification result for training case `SHOP-22`.

[Back to the Jupyter project index](../README.md)

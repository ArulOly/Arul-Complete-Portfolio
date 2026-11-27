# Jupyter: Access Review — Recovery for Public Dataset (Project 033518)

## Project overview

A synthetic public-data exercise provides the setting for this access review exercise. The embedded code is an AI-generated reference implementation based on the student guide and uses fictional data.

## Problem statement

The public dataset needs an inspectable notebook for access review and recovery.

## Project objectives

- Compare synthetic active grants with an approved role policy.
- Display the fallback result and record why the primary failed.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Jupyter, IPython rich display, Python standard library, and JSON.

## System architecture

`Local JSON → Python notebook cell → workflow validation → rich notebook display`

## Features

- Compare synthetic active grants with an approved role policy.
- Display the fallback result and record why the primary failed.

## Prerequisites

Install Python 3.10 or later and JupyterLab or Jupyter Notebook with a Python kernel. Use a text editor and a local practice directory.

## Complete Jupyter source code

```python
# Jupyter reference implementation for project 033518.


from IPython.display import JSON, display
from pathlib import Path
import json


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

`evaluate()` calculates the access review outcome. `analyze()` applies the recovery rules, and IPython renders the response inside the notebook.

## Sample input data

Save this synthetic input as `sample.json`.

```json
{
  "primary": {
    "asset": "DATA-12",
    "approved": [
      "read",
      "submit"
    ],
    "grants": [
      "read",
      "admin"
    ]
  },
  "fallback": {
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
}
```

## Execution instructions

Save the synthetic input as `sample.json` in the notebook folder. Open a new Python notebook in JupyterLab or Jupyter Notebook, paste the source block into one code cell, and run it with **Shift+Enter**. Restart the kernel before a clean repeat.

## Example output

Illustrative excerpt for the synthetic sample.

```json
{
  "excess_grants": [
    "admin"
  ]
}
```

## Testing and validation

Actual validation ran the embedded cell in a Jupyter Python kernel with temporary input files. The rendered result matched the expected field and repeat analysis produced the same value. A failed primary used its fallback. Two bad snapshots produced a validation error.

```json
{
  "excess_grants": [
    "admin"
  ]
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

Practice access review using Jupyter and check a repeatable recovery workflow in the public dataset scenario.

## Conclusion

The project records an observable access review result for training case `DATA-12`.

[Back to the Jupyter project index](../README.md)

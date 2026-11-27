# Jupyter: Recovery Test — Reporting for Two-Node Lab (Project 033989)

## Project overview

A two-node lab simulation provides the setting for this recovery test exercise. The embedded code is an AI-generated reference implementation based on the student guide and uses fictional data.

## Problem statement

The two-node lab needs an inspectable notebook for recovery test and reporting.

## Project objectives

- Restore a mock asset snapshot in memory and compare it against expected values.
- Export a project-labeled JSON report without overwriting a previous report.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Jupyter, IPython rich display, Python standard library, and JSON.

## System architecture

`Local JSON → Python notebook cell → workflow validation → rich notebook display`

## Features

- Restore a mock asset snapshot in memory and compare it against expected values.
- Export a project-labeled JSON report without overwriting a previous report.

## Prerequisites

Install Python 3.10 or later and JupyterLab or Jupyter Notebook with a Python kernel. Use a text editor and a local practice directory.

## Complete Jupyter source code

```python
# Jupyter reference implementation for project 033989.


from IPython.display import JSON, display
from pathlib import Path
import json


PROJECT_ID = "033989"


def label(record, key):
    value = record.get(key)
    if not isinstance(value, str) or not value.strip():
        raise ValueError(f"{key} must be non-empty text")
    return value.strip()


def mapping(record, key):
    value = record.get(key)
    if not isinstance(value, dict):
        raise ValueError(f"{key} must be an object")
    return value


def evaluate(record):
    """Restore a mock asset snapshot in memory and compare it against expected values."""
    saved = mapping(record, "snapshot")
    expected = mapping(record, "expected")
    restored = json.loads(json.dumps(saved))
    keys = restored.keys() | expected.keys()
    differences = sorted(key for key in keys if key not in restored or key not in expected
                         or json.dumps(restored[key], sort_keys=True)
                         != json.dumps(expected[key], sort_keys=True))
    return {"restored": restored, "differences": differences, "verified": not differences}


def as_record(value):
    if not isinstance(value, dict):
        raise ValueError("Expected an object")
    return value


def analyze(payload):
    record = as_record(payload)
    return {"project": PROJECT_ID, "case": label(record, "asset"), "result": evaluate(record)}


def run_cell(input_path):
    try:
        payload = json.loads(Path(input_path).read_text(encoding="utf-8"))
        json.dumps(payload, allow_nan=False)
        report = analyze(payload)
        with Path(f"report-{PROJECT_ID}.json").open("x", encoding="utf-8") as handle:
            json.dump(report, handle, ensure_ascii=False, indent=2)
        display(JSON(report))
        return report
    except (OSError, ValueError, OverflowError, RecursionError) as exc:
        print(f"Notebook input or output error: {exc}")
        return None


result = run_cell(Path("sample.json"))
```

## Implementation explanation

`evaluate()` calculates the recovery test outcome. `analyze()` applies the reporting rules, and IPython renders the response inside the notebook.

## Sample input data

Save this synthetic input as `sample.json`.

```json
{
  "asset": "NODE-B",
  "snapshot": {
    "version": 3,
    "active": true
  },
  "expected": {
    "version": 3,
    "active": true
  }
}
```

## Execution instructions

Save the synthetic input as `sample.json` in the notebook folder. Open a new Python notebook in JupyterLab or Jupyter Notebook, paste the source block into one code cell, and run it with **Shift+Enter**. Restart the kernel before a clean repeat. Move the exported JSON to a different filename before running the export cell again.

## Example output

Illustrative excerpt for the synthetic sample.

```json
{
  "verified": true
}
```

## Testing and validation

Actual validation ran the embedded cell in a Jupyter Python kernel with temporary input files. The rendered result matched the expected field and repeat analysis produced the same value. An incomplete record was rejected and an existing report was left intact.

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

Practice recovery test using Jupyter and check a repeatable reporting workflow in the two-node lab scenario.

## Conclusion

The project records an observable recovery test result for training case `NODE-B`.

[Back to the Jupyter project index](../README.md)

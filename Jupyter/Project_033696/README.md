# Jupyter: Configuration Comparison — Baseline Setup for Portfolio Demo (Project 033696)

## Project overview

A repeatable portfolio demonstration provides the setting for this configuration comparison exercise. This exercise is based on the student guide and uses fictional data. This project includes source code, sample data, and instructions to run it.

## Problem statement

The portfolio demo needs an inspectable notebook for configuration comparison and baseline setup.

## Project objectives

- Identify added, removed, and changed settings between two configuration objects.
- Display a reference calculation alongside the case label.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Jupyter, IPython rich display, Python standard library, and JSON.

## System architecture

`Local JSON → Python notebook cell → workflow validation → rich notebook display`

## Features

- Identify added, removed, and changed settings between two configuration objects.
- Display a reference calculation alongside the case label.

## Prerequisites

Install Python 3.10 or later and JupyterLab or Jupyter Notebook with a Python kernel. Use a text editor and a local practice directory.

## Complete Jupyter source code

```python
# Jupyter reference implementation for project 033696.


from IPython.display import JSON, display
from pathlib import Path
import json


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
    """Identify added, removed, and changed settings between two configuration objects."""
    baseline = mapping(record, "baseline")
    current = mapping(record, "current")
    return {"added": sorted(current.keys() - baseline.keys()),
            "removed": sorted(baseline.keys() - current.keys()),
            "changed": sorted(key for key in baseline.keys() & current.keys()
                              if json.dumps(baseline[key], sort_keys=True)
                              != json.dumps(current[key], sort_keys=True))}


def as_record(value):
    if not isinstance(value, dict):
        raise ValueError("Expected an object")
    return value


def analyze(payload):
    record = as_record(payload)
    return {"case": label(record, "asset"), "result": evaluate(record)}


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

`evaluate()` calculates the configuration comparison outcome. `analyze()` applies the baseline setup rules, and IPython renders the response inside the notebook.

## Sample input data

Save this synthetic input as `sample.json`.

```json
{
  "asset": "DEMO-02",
  "baseline": {
    "timeout": 30,
    "theme": "light"
  },
  "current": {
    "timeout": 45,
    "retries": 2
  }
}
```

## Execution instructions

Save the synthetic input as `sample.json` in the notebook folder. Open a new Python notebook in JupyterLab or Jupyter Notebook, paste the source block into one code cell, and run it with **Shift+Enter**. Restart the kernel before a clean repeat.

## Example output

Illustrative excerpt for the synthetic sample.

```json
{
  "changed": [
    "timeout"
  ]
}
```

## Testing and validation

Actual validation ran the embedded cell in a Jupyter Python kernel with temporary input files. The rendered result matched the expected field and repeat analysis produced the same value. An incomplete record produced a validation error.

```json
{
  "changed": [
    "timeout"
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

Practice configuration comparison using Jupyter and check a repeatable baseline setup workflow in the portfolio demo scenario.

## Conclusion

The project records an observable configuration comparison result for training case `DEMO-02`.

[Back to the Jupyter project index](../README.md)

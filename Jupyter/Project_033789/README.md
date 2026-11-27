# Jupyter: Data Validation — Reporting for Two-Node Lab (Project 033789)

## Project overview

A two-node lab simulation provides the setting for this data validation exercise. The embedded code is an AI-generated reference implementation based on the student guide and uses fictional data.

## Problem statement

The two-node lab needs an inspectable notebook for data validation and reporting.

## Project objectives

- Return accepted values and row-level diagnostics for invalid or duplicate codes.
- Export a project-labeled JSON report without overwriting a previous report.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Jupyter, IPython rich display, Python standard library, and JSON.

## System architecture

`Local JSON → Python notebook cell → workflow validation → rich notebook display`

## Features

- Return accepted values and row-level diagnostics for invalid or duplicate codes.
- Export a project-labeled JSON report without overwriting a previous report.

## Prerequisites

Install Python 3.10 or later and JupyterLab or Jupyter Notebook with a Python kernel. Use a text editor and a local practice directory.

## Complete Jupyter source code

```python
# Jupyter reference implementation for project 033789.


from IPython.display import JSON, display
from pathlib import Path
import json


PROJECT_ID = "033789"


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
    """Return accepted values and row-level diagnostics for invalid or duplicate codes."""
    accepted = {}
    rejected = []
    for position, row in enumerate(objects(record, "rows"), 1):
        try:
            code = label(row, "code").upper()
            count = integer(row, "count")
            if code in accepted:
                raise ValueError("duplicate code " + code)
            accepted[code] = count
        except ValueError as exc:
            rejected.append({"row": position, "error": str(exc)})
    return {"accepted": accepted, "rejected": rejected}


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

`evaluate()` calculates the data validation outcome. `analyze()` applies the reporting rules, and IPython renders the response inside the notebook.

## Sample input data

Save this synthetic input as `sample.json`.

```json
{
  "asset": "NODE-B",
  "rows": [
    {
      "code": "KIT-01",
      "count": 4
    },
    {
      "code": "KIT-01",
      "count": 2
    },
    {
      "code": "KIT-02",
      "count": -1
    }
  ]
}
```

## Execution instructions

Save the synthetic input as `sample.json` in the notebook folder. Open a new Python notebook in JupyterLab or Jupyter Notebook, paste the source block into one code cell, and run it with **Shift+Enter**. Restart the kernel before a clean repeat. Move the exported JSON to a different filename before running the export cell again.

## Example output

Illustrative excerpt for the synthetic sample.

```json
{
  "accepted": {
    "KIT-01": 4
  }
}
```

## Testing and validation

Actual validation ran the embedded cell in a Jupyter Python kernel with temporary input files. The rendered result matched the expected field and repeat analysis produced the same value. An incomplete record was rejected and an existing report was left intact.

```json
{
  "accepted": {
    "KIT-01": 4
  }
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

Practice data validation using Jupyter and check a repeatable reporting workflow in the two-node lab scenario.

## Conclusion

The project records an observable data validation result for training case `NODE-B`.

[Back to the Jupyter project index](../README.md)

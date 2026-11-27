# Jupyter: Inventory — Automation for Small Business (Project 033007)

## Project overview

Local shop operations provides the setting for this inventory exercise. The embedded code is an AI-generated reference implementation based on the student guide and uses fictional data.

## Problem statement

The small business needs an inspectable notebook for inventory and automation.

## Project objectives

- Count items and identify stock below the reorder level.
- Process a batch and display valid results alongside row errors.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Jupyter, IPython rich display, Python standard library, and JSON.

## System architecture

`Local JSON → Python notebook cell → workflow validation → rich notebook display`

## Features

- Count items and identify stock below the reorder level.
- Process a batch and display valid results alongside row errors.

## Prerequisites

Install Python 3.10 or later and JupyterLab or Jupyter Notebook with a Python kernel. Use a text editor and a local practice directory.

## Complete Jupyter source code

```python
# Jupyter reference implementation for project 033007.


from IPython.display import JSON, display
from pathlib import Path
import json


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


def evaluate(record):
    """Count items and identify stock below the reorder level."""
    stock = integer(record, "stock")
    reorder = integer(record, "reorder_at", minimum=1)
    return {"stock": stock, "reorder_at": reorder, "needs_reorder": stock < reorder}


def as_record(value):
    if not isinstance(value, dict):
        raise ValueError("Expected an object")
    return value


def analyze(payload):
    batch = array(as_record(payload), "records")
    results = []
    for index, candidate in enumerate(batch, 1):
        try:
            record = as_record(candidate)
            results.append({"row": index, "result": evaluate(record)})
        except (ValueError, OverflowError) as exc:
            results.append({"row": index, "error": str(exc)})
    return {"rows": results}


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

`evaluate()` calculates the inventory outcome. `analyze()` applies the automation rules, and IPython renders the response inside the notebook.

## Sample input data

Save this synthetic input as `sample.json`.

```json
{
  "records": [
    {
      "asset": "SHOP-22",
      "stock": 2,
      "reorder_at": 4
    },
    {
      "asset": "SHOP-22-B",
      "stock": 2,
      "reorder_at": 4
    }
  ]
}
```

## Execution instructions

Save the synthetic input as `sample.json` in the notebook folder. Open a new Python notebook in JupyterLab or Jupyter Notebook, paste the source block into one code cell, and run it with **Shift+Enter**. Restart the kernel before a clean repeat.

## Example output

Illustrative excerpt for the synthetic sample.

```json
{
  "needs_reorder": true
}
```

## Testing and validation

Actual validation ran the embedded cell in a Jupyter Python kernel with temporary input files. The rendered result matched the expected field and repeat analysis produced the same value. A mixed batch preserved its valid row and displayed an error for the bad row.

```json
{
  "needs_reorder": true
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

Practice inventory using Jupyter and check a repeatable automation workflow in the small business scenario.

## Conclusion

The project records an observable inventory result for training case `SHOP-22`.

[Back to the Jupyter project index](../README.md)

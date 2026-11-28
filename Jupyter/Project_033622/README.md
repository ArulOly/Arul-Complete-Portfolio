# Jupyter: Service Check — Automation for Branch Office Simulator (Project 033622)

## Project overview

A simulated branch office provides the setting for this service check exercise. The embedded code is a reference implementation based on the student guide and uses fictional data.

## Problem statement

The branch office simulator needs an inspectable notebook for service check and automation.

## Project objectives

- Classify synthetic service probes by status and latency.
- Process a batch and display valid results alongside row errors.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Jupyter, IPython rich display, Python standard library, and JSON.

## System architecture

`Local JSON → Python notebook cell → workflow validation → rich notebook display`

## Features

- Classify synthetic service probes by status and latency.
- Process a batch and display valid results alongside row errors.

## Prerequisites

Install Python 3.10 or later and JupyterLab or Jupyter Notebook with a Python kernel. Use a text editor and a local practice directory.

## Complete Jupyter source code

```python
# Jupyter reference implementation for project 033622.


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

`evaluate()` calculates the service check outcome. `analyze()` applies the automation rules, and IPython renders the response inside the notebook.

## Sample input data

Save this synthetic input as `sample.json`.

```json
{
  "records": [
    {
      "asset": "BRANCH-08",
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
      "asset": "BRANCH-08-B",
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

Save the synthetic input as `sample.json` in the notebook folder. Open a new Python notebook in JupyterLab or Jupyter Notebook, paste the source block into one code cell, and run it with **Shift+Enter**. Restart the kernel before a clean repeat.

## Example output

Illustrative excerpt for the synthetic sample.

```json
{
  "healthy": 1
}
```

## Testing and validation

Actual validation ran the embedded cell in a Jupyter Python kernel with temporary input files. The rendered result matched the expected field and repeat analysis produced the same value. A mixed batch preserved its valid row and displayed an error for the bad row.

```json
{
  "healthy": 1
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

Practice service check using Jupyter and check a repeatable automation workflow in the branch office simulator scenario.

## Conclusion

The project records an observable service check result for training case `BRANCH-08`.

[Back to the Jupyter project index](../README.md)

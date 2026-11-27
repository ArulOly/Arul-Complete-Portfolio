# Jupyter: Automation — Recovery for Portfolio Demo (Project 033898)

## Project overview

A repeatable portfolio demonstration provides the setting for this automation exercise. The embedded code is an AI-generated reference implementation based on the student guide and uses fictional data.

## Problem statement

The portfolio demo needs an inspectable notebook for automation and recovery.

## Project objectives

- Resolve a local task plan and propagate prerequisite failures to dependent work.
- Display the fallback result and record why the primary failed.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Jupyter, IPython rich display, Python standard library, and JSON.

## System architecture

`Local JSON → Python notebook cell → workflow validation → rich notebook display`

## Features

- Resolve a local task plan and propagate prerequisite failures to dependent work.
- Display the fallback result and record why the primary failed.

## Prerequisites

Install Python 3.10 or later and JupyterLab or Jupyter Notebook with a Python kernel. Use a text editor and a local practice directory.

## Complete Jupyter source code

```python
# Jupyter reference implementation for project 033898.


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


def objects(record, key):
    values = array(record, key)
    if any(not isinstance(value, dict) for value in values):
        raise ValueError(f"every item in {key} must be an object")
    return values


def labels(record, key, allow_empty=False):
    return [label({key: item}, key) for item in array(record, key, allow_empty)]


def evaluate(record):
    """Resolve a local task plan and propagate prerequisite failures to dependent work."""
    pending = {}
    for task in objects(record, "tasks"):
        name = label(task, "name")
        needs = labels(task, "needs", allow_empty=True)
        ready = task.get("ready")
        if type(ready) is not bool or name in pending:
            raise ValueError("each task needs a unique name and Boolean ready flag")
        pending[name] = (needs, ready)
    known = set(pending)
    if any(set(needs) - known for needs, _ in pending.values()):
        raise ValueError("a prerequisite task is missing")
    states = {}
    while pending:
        eligible = [name for name, (needs, _) in pending.items() if set(needs) <= states.keys()]
        if not eligible:
            raise ValueError("task dependencies contain a cycle")
        for name in eligible:
            needs, ready = pending.pop(name)
            if any(states[need] != "done" for need in needs):
                states[name] = "blocked"
            else:
                states[name] = "done" if ready else "failed"
    return {"task_states": states}


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

`evaluate()` calculates the automation outcome. `analyze()` applies the recovery rules, and IPython renders the response inside the notebook.

## Sample input data

Save this synthetic input as `sample.json`.

```json
{
  "primary": {
    "asset": "DEMO-02",
    "tasks": [
      {
        "name": "collect",
        "ready": true,
        "needs": []
      },
      {
        "name": "verify",
        "ready": false,
        "needs": [
          "collect"
        ]
      },
      {
        "name": "publish",
        "ready": true,
        "needs": [
          "verify"
        ]
      }
    ]
  },
  "fallback": {
    "asset": "DEMO-02",
    "tasks": [
      {
        "name": "collect",
        "ready": true,
        "needs": []
      },
      {
        "name": "verify",
        "ready": false,
        "needs": [
          "collect"
        ]
      },
      {
        "name": "publish",
        "ready": true,
        "needs": [
          "verify"
        ]
      }
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
  "task_states": {
    "collect": "done",
    "verify": "failed",
    "publish": "blocked"
  }
}
```

## Testing and validation

Actual validation ran the embedded cell in a Jupyter Python kernel with temporary input files. The rendered result matched the expected field and repeat analysis produced the same value. A failed primary used its fallback. Two bad snapshots produced a validation error.

```json
{
  "task_states": {
    "collect": "done",
    "verify": "failed",
    "publish": "blocked"
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

Practice automation using Jupyter and check a repeatable recovery workflow in the portfolio demo scenario.

## Conclusion

The project records an observable automation result for training case `DEMO-02`.

[Back to the Jupyter project index](../README.md)

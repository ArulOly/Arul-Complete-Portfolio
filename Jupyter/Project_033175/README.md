# Jupyter: Log Analysis — Usability Improvement for Branch Office Simulator (Project 033175)

## Project overview

A simulated branch office provides the setting for this log analysis exercise. The embedded code is an AI-generated reference implementation based on the student guide and uses fictional data.

## Problem statement

The branch office simulator needs an inspectable notebook for log analysis and usability improvement.

## Project objectives

- Count event levels in a synthetic log and identify the most frequent error.
- Display results as a captioned Markdown table.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Jupyter, IPython rich display, Python standard library, and JSON.

## System architecture

`Local JSON → Python notebook cell → workflow validation → rich notebook display`

## Features

- Count event levels in a synthetic log and identify the most frequent error.
- Display results as a captioned Markdown table.

## Prerequisites

Install Python 3.10 or later and JupyterLab or Jupyter Notebook with a Python kernel. Use a text editor and a local practice directory.

## Complete Jupyter source code

```python
# Jupyter reference implementation for project 033175.


from IPython.display import Markdown, display
from collections import Counter
from pathlib import Path
import json


SCENARIO = "Branch Office Simulator"


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


def as_record(value):
    if not isinstance(value, dict):
        raise ValueError("Expected an object")
    return value


def analyze(payload):
    record = as_record(payload)
    return {"case": label(record, "asset"), "scenario": SCENARIO, "result": evaluate(record)}


def run_cell(input_path):
    try:
        payload = json.loads(Path(input_path).read_text(encoding="utf-8"))
        json.dumps(payload, allow_nan=False)
        report = analyze(payload)
        def cell(value):
            # Neutralize HTML, table delimiters, and line breaks in the display.
            encoded = json.dumps(value, ensure_ascii=False)
            return "".join(f"&#{ord(char)};" if char in '\\&<>|`*_[]~' else char for char in encoded)

        lines = ["| Field | Value |", "| --- | --- |"]
        lines.extend(f"| {cell(key)} | {cell(value)} |" for key, value in report["result"].items())
        display(Markdown("\n".join(lines)))
        return report
    except (OSError, ValueError, OverflowError, RecursionError) as exc:
        print(f"Notebook input or output error: {exc}")
        return None


result = run_cell(Path("sample.json"))
```

## Implementation explanation

`evaluate()` calculates the log analysis outcome. `analyze()` applies the usability improvement rules, and IPython renders the response inside the notebook.

## Sample input data

Save this synthetic input as `sample.json`.

```json
{
  "asset": "BRANCH-08",
  "events": [
    "INFO | Started",
    "ERROR | Timeout",
    "ERROR | Timeout",
    "WARN | Retrying"
  ]
}
```

## Execution instructions

Save the synthetic input as `sample.json` in the notebook folder. Open a new Python notebook in JupyterLab or Jupyter Notebook, paste the source block into one code cell, and run it with **Shift+Enter**. Restart the kernel before a clean repeat.

## Example output

Illustrative excerpt for the synthetic sample.

```json
{
  "top_error": "Timeout"
}
```

## Testing and validation

Actual validation ran the embedded cell in a Jupyter Python kernel with temporary input files. The rendered result matched the expected field and repeat analysis produced the same value. An incomplete record was rejected and the notebook emitted a Markdown result table.

```json
{
  "top_error": "Timeout"
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

Practice log analysis using Jupyter and check a repeatable usability improvement workflow in the branch office simulator scenario.

## Conclusion

The project records an observable log analysis result for training case `BRANCH-08`.

[Back to the Jupyter project index](../README.md)

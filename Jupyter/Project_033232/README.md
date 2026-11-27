# Jupyter: Backup Verification — Automation for Single-Host Lab (Project 033232)

## Project overview

One simulated lab host provides the setting for this backup verification exercise. The embedded code is an AI-generated reference implementation based on the student guide and uses fictional data.

## Problem statement

The single-host lab needs an inspectable notebook for backup verification and automation.

## Project objectives

- Compare a synthetic backup payload against an expected SHA-256 digest.
- Process a batch and display valid results alongside row errors.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Jupyter, IPython rich display, Python standard library, and JSON.

## System architecture

`Local JSON → Python notebook cell → workflow validation → rich notebook display`

## Features

- Compare a synthetic backup payload against an expected SHA-256 digest.
- Process a batch and display valid results alongside row errors.

## Prerequisites

Install Python 3.10 or later and JupyterLab or Jupyter Notebook with a Python kernel. Use a text editor and a local practice directory.

## Complete Jupyter source code

```python
# Jupyter reference implementation for project 033232.


from IPython.display import JSON, display
from pathlib import Path
import hashlib
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

`evaluate()` calculates the backup verification outcome. `analyze()` applies the automation rules, and IPython renders the response inside the notebook.

## Sample input data

Save this synthetic input as `sample.json`.

```json
{
  "records": [
    {
      "asset": "HOST-03",
      "payload": "Backup manifest for HOST-03, revision 1",
      "sha256": "0bfdb55001e342f5b774611eb390453518709f6ac3c57205f1d78c275f2ffc9b"
    },
    {
      "asset": "HOST-03-B",
      "payload": "Backup manifest for HOST-03, revision 1",
      "sha256": "0bfdb55001e342f5b774611eb390453518709f6ac3c57205f1d78c275f2ffc9b"
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
  "verified": true
}
```

## Testing and validation

Actual validation ran the embedded cell in a Jupyter Python kernel with temporary input files. The rendered result matched the expected field and repeat analysis produced the same value. A mixed batch preserved its valid row and displayed an error for the bad row.

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

Practice backup verification using Jupyter and check a repeatable automation workflow in the single-host lab scenario.

## Conclusion

The project records an observable backup verification result for training case `HOST-03`.

[Back to the Jupyter project index](../README.md)

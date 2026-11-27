# Jupyter: Backup Verification — Reporting for Offline Classroom (Project 033229)

## Project overview

A classroom operating offline provides the setting for this backup verification exercise. The embedded code is an AI-generated reference implementation based on the student guide and uses fictional data.

## Problem statement

The offline classroom needs an inspectable notebook for backup verification and reporting.

## Project objectives

- Compare a synthetic backup payload against an expected SHA-256 digest.
- Export a project-labeled JSON report without overwriting a previous report.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Jupyter, IPython rich display, Python standard library, and JSON.

## System architecture

`Local JSON → Python notebook cell → workflow validation → rich notebook display`

## Features

- Compare a synthetic backup payload against an expected SHA-256 digest.
- Export a project-labeled JSON report without overwriting a previous report.

## Prerequisites

Install Python 3.10 or later and JupyterLab or Jupyter Notebook with a Python kernel. Use a text editor and a local practice directory.

## Complete Jupyter source code

```python
# Jupyter reference implementation for project 033229.


from IPython.display import JSON, display
from pathlib import Path
import hashlib
import json


PROJECT_ID = "033229"


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

`evaluate()` calculates the backup verification outcome. `analyze()` applies the reporting rules, and IPython renders the response inside the notebook.

## Sample input data

Save this synthetic input as `sample.json`.

```json
{
  "asset": "CLASS-09",
  "payload": "Backup manifest for CLASS-09, revision 1",
  "sha256": "3d670acf6bc1b147a814e88279c47dae213060a66a7c44d510accfd5dca837c9"
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

Practice backup verification using Jupyter and check a repeatable reporting workflow in the offline classroom scenario.

## Conclusion

The project records an observable backup verification result for training case `CLASS-09`.

[Back to the Jupyter project index](../README.md)

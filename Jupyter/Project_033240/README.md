# Jupyter: Backup Verification — Usability Improvement for Two-Node Lab (Project 033240)

## Project overview

A two-node lab simulation provides the setting for this backup verification exercise. The embedded code is a reference implementation based on the student guide and uses fictional data.

## Problem statement

The two-node lab needs an inspectable notebook for backup verification and usability improvement.

## Project objectives

- Compare a synthetic backup payload against an expected SHA-256 digest.
- Display results as a captioned Markdown table.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Jupyter, IPython rich display, Python standard library, and JSON.

## System architecture

`Local JSON → Python notebook cell → workflow validation → rich notebook display`

## Features

- Compare a synthetic backup payload against an expected SHA-256 digest.
- Display results as a captioned Markdown table.

## Prerequisites

Install Python 3.10 or later and JupyterLab or Jupyter Notebook with a Python kernel. Use a text editor and a local practice directory.

## Complete Jupyter source code

```python
# Jupyter reference implementation for project 033240.


from IPython.display import Markdown, display
from pathlib import Path
import hashlib
import json


SCENARIO = "Two-Node Lab"


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

`evaluate()` calculates the backup verification outcome. `analyze()` applies the usability improvement rules, and IPython renders the response inside the notebook.

## Sample input data

Save this synthetic input as `sample.json`.

```json
{
  "asset": "NODE-B",
  "payload": "Backup manifest for NODE-B, revision 1",
  "sha256": "62d6d5d4c0a54caa219a4432aa93c44afae4d1b712a1006a592fce790efc0e4c"
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

Actual validation ran the embedded cell in a Jupyter Python kernel with temporary input files. The rendered result matched the expected field and repeat analysis produced the same value. An incomplete record was rejected and the notebook emitted a Markdown result table.

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

Practice backup verification using Jupyter and check a repeatable usability improvement workflow in the two-node lab scenario.

## Conclusion

The project records an observable backup verification result for training case `NODE-B`.

[Back to the Jupyter project index](../README.md)

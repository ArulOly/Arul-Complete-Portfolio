# FastAPI: Recovery Test — Reporting for Home Development (Project 049964)

## Project overview

A home development workspace provides the setting for this recovery test exercise. The embedded code is a reference implementation based on the student guide and uses fictional data.

## Problem statement

The home development needs a consistent HTTP interface for recovery test and reporting.

## Project objectives

- Restore a mock asset snapshot in memory and compare it against expected values.
- Return the result as a downloadable JSON attachment.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Python, FastAPI, JSON, local HTTP, and the Python standard library.

## System architecture

`JSON POST → request validation → workflow → HTTP JSON response`

## Features

- Restore a mock asset snapshot in memory and compare it against expected values.
- Return the result as a downloadable JSON attachment.

## Prerequisites

Install Python 3.10 or later and use curl for the example request. Use a text editor and a local practice directory.

## Complete FastAPI source code

```python
"""Local FastAPI reference implementation for project 049964."""

import json
import math

from fastapi import FastAPI, Request
from fastapi.responses import JSONResponse

app = FastAPI()

PROJECT_ID = "049964"


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


def parse_json(data):
    """Reject non-finite numbers before passing a record to the workflow."""
    def reject_constant(token):
        raise ValueError(f"Unsupported JSON constant: {token}")

    def finite_number(token):
        value = float(token)
        if not math.isfinite(value):
            raise ValueError("JSON numbers must be finite")
        return value

    return json.loads(data, parse_constant=reject_constant, parse_float=finite_number)


def as_object(payload):
    if not isinstance(payload, dict):
        raise ValueError("Expected a JSON object")
    return payload


def process(payload):
    record = as_object(payload)
    return {"project": PROJECT_ID, "asset": label(record, "asset"), "result": evaluate(record)}, 200


@app.post("/evaluate")
async def evaluate_route(request: Request):
    if request.headers.get("content-type", "").split(";")[0].strip().lower() != "application/json":
        return JSONResponse({"error": "Send application/json"}, status_code=415)
    try:
        payload = parse_json(await request.body())
        body, status = process(payload)
    except (ValueError, OverflowError, RecursionError) as exc:
        return JSONResponse({"error": str(exc)}, status_code=400)
    return JSONResponse(body, status_code=status, headers={"Content-Disposition": f"attachment; filename=report-{PROJECT_ID}.json"})
```

## Implementation explanation

`evaluate()` applies the recovery test rules to one record. `process()` implements reporting. The route handles media type, parsing, and status codes.

## Sample input data

Save this synthetic request as `sample.json`.

```json
{
  "asset": "DEV-05",
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

Save the code as `app.py` and the sample as `sample.json`. In a local virtual environment, install the dependency and start the server.

```sh
python -m pip install fastapi uvicorn
python -m uvicorn app:app --host 127.0.0.1 --port 8000
```

From another terminal in the same directory, submit the sample.

```sh
curl -H "Content-Type: application/json" --data-binary @sample.json http://127.0.0.1:8000/evaluate
```

Stop the server with Ctrl+C when finished.

## Example output

Illustrative excerpt for the synthetic sample.

```json
{
  "verified": true
}
```

## Testing and validation

Actual validation used the FastAPI test client and synthetic requests. The sample returned HTTP 200 twice with identical JSON. The test observed the field below. An incomplete record returned HTTP 400. The response included the project-specific attachment filename. Malformed JSON and non-finite values returned HTTP 400 and the wrong media type returned HTTP 415.

```json
{
  "verified": true
}
```

## Error handling

The route reports malformed requests with HTTP 400 and unsupported media types with HTTP 415. Return the result as a downloadable JSON attachment.

## Troubleshooting

- Confirm the documented runtime and copy the complete source block.
- Start with the documented sample and use the filename or request path shown above.
- Compare the observed value with the example before changing an input.

## Limitations

This local training API works with synthetic records and does not persist data across requests.

## Learning outcomes

Practice recovery test using FastAPI and check a repeatable reporting workflow in the home development scenario.

## Conclusion

The project records an observable recovery test result for training case `DEV-05`.

[Back to the FastAPI project index](../README.md)

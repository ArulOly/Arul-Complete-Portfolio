# FastAPI: Performance Baseline — Recovery for Single-Host Lab (Project 049583)

## Project overview

One simulated lab host provides the setting for this performance baseline exercise. The embedded code is a reference implementation based on the student guide and uses fictional data.

## Problem statement

The single-host lab needs a consistent HTTP interface for performance baseline and recovery.

## Project objectives

- Calculate median and nearest-rank p95 latency against a stated budget.
- Evaluate a fallback record when the primary input fails validation.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Python, FastAPI, JSON, local HTTP, and the Python standard library.

## System architecture

`JSON POST → request validation → workflow → HTTP JSON response`

## Features

- Calculate median and nearest-rank p95 latency against a stated budget.
- Evaluate a fallback record when the primary input fails validation.

## Prerequisites

Install Python 3.10 or later and use curl for the example request. Use a text editor and a local practice directory.

## Complete FastAPI source code

```python
"""Local FastAPI reference implementation for project 049583."""

import json
import math
import statistics

from fastapi import FastAPI, Request
from fastapi.responses import JSONResponse

app = FastAPI()


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


def series(record, key, maximum=1_000_000_000):
    return [integer({key: item}, key, maximum=maximum) for item in array(record, key)]


def evaluate(record):
    """Calculate median and nearest-rank p95 latency against a stated budget."""
    readings = sorted(series(record, "latencies_ms"))
    rank = (95 * len(readings) + 99) // 100
    p95 = readings[rank - 1]
    budget = integer(record, "budget_ms", minimum=1)
    return {"samples": len(readings), "median_ms": statistics.median(readings),
            "p95_ms": p95, "within_budget": p95 <= budget}


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
    envelope = as_object(payload)
    try:
        record = as_object(envelope.get("primary"))
        return {"source": "primary", "result": evaluate(record)}, 200
    except (ValueError, OverflowError) as exc:
        record = as_object(envelope.get("fallback"))
        return {"source": "fallback", "primary_error": str(exc), "result": evaluate(record)}, 200


@app.post("/evaluate")
async def evaluate_route(request: Request):
    if request.headers.get("content-type", "").split(";")[0].strip().lower() != "application/json":
        return JSONResponse({"error": "Send application/json"}, status_code=415)
    try:
        payload = parse_json(await request.body())
        body, status = process(payload)
    except (ValueError, OverflowError, RecursionError) as exc:
        return JSONResponse({"error": str(exc)}, status_code=400)
    return JSONResponse(body, status_code=status)
```

## Implementation explanation

`evaluate()` applies the performance baseline rules to one record. `process()` implements recovery. The route handles media type, parsing, and status codes.

## Sample input data

Save this synthetic request as `sample.json`.

```json
{
  "primary": {
    "asset": "HOST-03",
    "latencies_ms": [
      12,
      15,
      14,
      44,
      17
    ],
    "budget_ms": 30
  },
  "fallback": {
    "asset": "HOST-03",
    "latencies_ms": [
      12,
      15,
      14,
      44,
      17
    ],
    "budget_ms": 30
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
  "p95_ms": 44
}
```

## Testing and validation

Actual validation used the FastAPI test client and synthetic requests. The sample returned HTTP 200 twice with identical JSON. The test observed the field below. An incomplete primary used the fallback, and invalid primary and fallback records returned HTTP 400. Malformed JSON and non-finite values returned HTTP 400 and the wrong media type returned HTTP 415.

```json
{
  "p95_ms": 44
}
```

## Error handling

The route reports malformed requests with HTTP 400 and unsupported media types with HTTP 415. Evaluate a fallback record when the primary input fails validation.

## Troubleshooting

- Confirm the documented runtime and copy the complete source block.
- Start with the documented sample and use the filename or request path shown above.
- Compare the observed value with the example before changing an input.

## Limitations

This local training API works with synthetic records and does not persist data across requests.

## Learning outcomes

Practice performance baseline using FastAPI and check a repeatable recovery workflow in the single-host lab scenario.

## Conclusion

The project records an observable performance baseline result for training case `HOST-03`.

[Back to the FastAPI project index](../README.md)

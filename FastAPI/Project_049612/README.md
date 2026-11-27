# FastAPI: Service Check — Automation for Home Development (Project 049612)

## Project overview

A home development workspace provides the setting for this service check exercise. The embedded code is an AI-generated reference implementation based on the student guide and uses fictional data.

## Problem statement

The home development needs a consistent HTTP interface for service check and automation.

## Project objectives

- Classify synthetic service probes by status and latency.
- Evaluate a batch while keeping each valid result and each row error.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Python, FastAPI, JSON, local HTTP, and the Python standard library.

## System architecture

`JSON POST → request validation → workflow → HTTP JSON response`

## Features

- Classify synthetic service probes by status and latency.
- Evaluate a batch while keeping each valid result and each row error.

## Prerequisites

Install Python 3.10 or later and use curl for the example request. Use a text editor and a local practice directory.

## Complete FastAPI source code

```python
"""Local FastAPI reference implementation for project 049612."""

import json

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


def as_object(payload):
    if not isinstance(payload, dict):
        raise ValueError("Expected a JSON object")
    return payload


def process(payload):
    records = array(as_object(payload), "records")
    rows = []
    for index, candidate in enumerate(records, 1):
        try:
            record = as_object(candidate)
            rows.append({"row": index, "asset": label(record, "asset"), "result": evaluate(record)})
        except (ValueError, OverflowError) as exc:
            rows.append({"row": index, "error": str(exc)})
    return {"rows": rows, "failed": sum("error" in row for row in rows)}, 200


@app.post("/evaluate")
async def evaluate_route(request: Request):
    if request.headers.get("content-type", "").split(";")[0].strip().lower() != "application/json":
        return JSONResponse({"error": "Send application/json"}, status_code=415)
    try:
        payload = json.loads(await request.body())
        body, status = process(payload)
    except (ValueError, OverflowError, RecursionError) as exc:
        return JSONResponse({"error": str(exc)}, status_code=400)
    return JSONResponse(body, status_code=status)
```

## Implementation explanation

`evaluate()` applies the service check rules to one record. `process()` implements automation. The route handles media type, parsing, and status codes.

## Sample input data

Save this synthetic request as `sample.json`.

```json
{
  "records": [
    {
      "asset": "DEV-05",
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
      "asset": "DEV-05-B",
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
  "healthy": 1
}
```

## Testing and validation

Actual validation used the FastAPI test client and synthetic requests. The sample returned HTTP 200 twice with identical JSON. The test observed the field below. A mixed batch kept its valid row and reported one invalid row. Malformed JSON returned HTTP 400 and the wrong media type returned HTTP 415.

```json
{
  "healthy": 1
}
```

## Error handling

The route reports malformed requests with HTTP 400 and unsupported media types with HTTP 415. Evaluate a batch while keeping each valid result and each row error.

## Troubleshooting

- Confirm the documented runtime and copy the complete source block.
- Start with the documented sample and use the filename or request path shown above.
- Compare the observed value with the example before changing an input.

## Limitations

This local training API works with synthetic records and does not persist data across requests.

## Learning outcomes

Practice service check using FastAPI and check a repeatable automation workflow in the home development scenario.

## Conclusion

The project records an observable service check result for training case `DEV-05`.

[Back to the FastAPI project index](../README.md)

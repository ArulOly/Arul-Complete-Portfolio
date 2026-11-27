# FastAPI: Configuration Comparison — Recovery for Two-Node Lab (Project 049688)

## Project overview

A two-node lab simulation provides the setting for this configuration comparison exercise. The embedded code is an AI-generated reference implementation based on the student guide and uses fictional data.

## Problem statement

The two-node lab needs a consistent HTTP interface for configuration comparison and recovery.

## Project objectives

- Identify added, removed, and changed settings between two configuration objects.
- Evaluate a fallback record when the primary input fails validation.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Python, FastAPI, JSON, local HTTP, and the Python standard library.

## System architecture

`JSON POST → request validation → workflow → HTTP JSON response`

## Features

- Identify added, removed, and changed settings between two configuration objects.
- Evaluate a fallback record when the primary input fails validation.

## Prerequisites

Install Python 3.10 or later and use curl for the example request. Use a text editor and a local practice directory.

## Complete FastAPI source code

```python
"""Local FastAPI reference implementation for project 049688."""

import json

from fastapi import FastAPI, Request
from fastapi.responses import JSONResponse

app = FastAPI()


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
    """Identify added, removed, and changed settings between two configuration objects."""
    baseline = mapping(record, "baseline")
    current = mapping(record, "current")
    return {"added": sorted(current.keys() - baseline.keys()),
            "removed": sorted(baseline.keys() - current.keys()),
            "changed": sorted(key for key in baseline.keys() & current.keys()
                              if json.dumps(baseline[key], sort_keys=True)
                              != json.dumps(current[key], sort_keys=True))}


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
        payload = json.loads(await request.body())
        body, status = process(payload)
    except (ValueError, OverflowError, RecursionError) as exc:
        return JSONResponse({"error": str(exc)}, status_code=400)
    return JSONResponse(body, status_code=status)
```

## Implementation explanation

`evaluate()` applies the configuration comparison rules to one record. `process()` implements recovery. The route handles media type, parsing, and status codes.

## Sample input data

Save this synthetic request as `sample.json`.

```json
{
  "primary": {
    "asset": "NODE-B",
    "baseline": {
      "timeout": 30,
      "theme": "light"
    },
    "current": {
      "timeout": 45,
      "retries": 2
    }
  },
  "fallback": {
    "asset": "NODE-B",
    "baseline": {
      "timeout": 30,
      "theme": "light"
    },
    "current": {
      "timeout": 45,
      "retries": 2
    }
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
  "changed": [
    "timeout"
  ]
}
```

## Testing and validation

Actual validation used the FastAPI test client and synthetic requests. The sample returned HTTP 200 twice with identical JSON. The test observed the field below. An incomplete primary used the fallback, and invalid primary and fallback records returned HTTP 400. Malformed JSON returned HTTP 400 and the wrong media type returned HTTP 415.

```json
{
  "changed": [
    "timeout"
  ]
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

Practice configuration comparison using FastAPI and check a repeatable recovery workflow in the two-node lab scenario.

## Conclusion

The project records an observable configuration comparison result for training case `NODE-B`.

[Back to the FastAPI project index](../README.md)

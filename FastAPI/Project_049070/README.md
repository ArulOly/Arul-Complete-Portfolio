# FastAPI: Monitoring — Usability Improvement for Public Dataset (Project 049070)

## Project overview

A synthetic public-data exercise provides the setting for this monitoring exercise. The embedded code is an AI-generated reference implementation based on the student guide and uses fictional data.

## Problem statement

The public dataset needs a consistent HTTP interface for monitoring and usability improvement.

## Project objectives

- Summarize utilization readings and count readings above the limit.
- Offer a concise response and a GET route that explains the request format.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Python, FastAPI, JSON, local HTTP, and the Python standard library.

## System architecture

`JSON POST → request validation → workflow → HTTP JSON response`

## Features

- Summarize utilization readings and count readings above the limit.
- Offer a concise response and a GET route that explains the request format.

## Prerequisites

Install Python 3.10 or later and use curl for the example request. Use a text editor and a local practice directory.

## Complete FastAPI source code

```python
"""Local FastAPI reference implementation for project 049070."""

import json
import statistics

from fastapi import FastAPI, Request
from fastapi.responses import JSONResponse

app = FastAPI()

PROJECT_ID = "049070"


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
    """Summarize utilization readings and count readings above the limit."""
    readings = series(record, "readings_pct", maximum=100)
    limit = integer(record, "limit_pct", maximum=100)
    return {"mean_pct": round(statistics.mean(readings), 2),
            "peak_pct": max(readings), "above_limit": sum(n > limit for n in readings)}


def as_object(payload):
    if not isinstance(payload, dict):
        raise ValueError("Expected a JSON object")
    return payload


def process(payload):
    record = as_object(payload)
    return {"asset": label(record, "asset"), "result": evaluate(record)}, 200


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

@app.get("/evaluate")
def show_help():
    return {"method": "POST", "content_type": "application/json", "project": PROJECT_ID}
```

## Implementation explanation

`evaluate()` applies the monitoring rules to one record. `process()` implements usability improvement. The route handles media type, parsing, and status codes.

## Sample input data

Save this synthetic request as `sample.json`.

```json
{
  "asset": "DATA-12",
  "readings_pct": [
    35,
    81,
    54
  ],
  "limit_pct": 80
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
  "above_limit": 1
}
```

## Testing and validation

Actual validation used the FastAPI test client and synthetic requests. The sample returned HTTP 200 twice with identical JSON. The test observed the field below. An incomplete record returned HTTP 400. The help route returned the expected HTTP method. Malformed JSON returned HTTP 400 and the wrong media type returned HTTP 415.

```json
{
  "above_limit": 1
}
```

## Error handling

The route reports malformed requests with HTTP 400 and unsupported media types with HTTP 415. Offer a concise response and a GET route that explains the request format.

## Troubleshooting

- Confirm the documented runtime and copy the complete source block.
- Start with the documented sample and use the filename or request path shown above.
- Compare the observed value with the example before changing an input.

## Limitations

This local training API works with synthetic records and does not persist data across requests.

## Learning outcomes

Practice monitoring using FastAPI and check a repeatable usability improvement workflow in the public dataset scenario.

## Conclusion

The project records an observable monitoring result for training case `DATA-12`.

[Back to the FastAPI project index](../README.md)

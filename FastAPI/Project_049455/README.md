# FastAPI: Scheduled Reporting — Usability Improvement for School Lab (Project 049455)

## Project overview

Shared classroom equipment provides the setting for this scheduled reporting exercise. The embedded code is an AI-generated reference implementation based on the student guide and uses fictional data.

## Problem statement

The school lab needs a consistent HTTP interface for scheduled reporting and usability improvement.

## Project objectives

- Calculate the next due date from an explicit reporting interval and reference date.
- Offer a concise response and a GET route that explains the request format.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Python, FastAPI, JSON, local HTTP, and the Python standard library.

## System architecture

`JSON POST → request validation → workflow → HTTP JSON response`

## Features

- Calculate the next due date from an explicit reporting interval and reference date.
- Offer a concise response and a GET route that explains the request format.

## Prerequisites

Install Python 3.10 or later and use curl for the example request. Use a text editor and a local practice directory.

## Complete FastAPI source code

```python
"""Local FastAPI reference implementation for project 049455."""

from datetime import date
from datetime import timedelta
import json
import math

from fastapi import FastAPI, Request
from fastapi.responses import JSONResponse

app = FastAPI()

PROJECT_ID = "049455"


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


def iso_date(record, key):
    value = label(record, key)
    try:
        parsed = date.fromisoformat(value)
    except ValueError as exc:
        raise ValueError(f"{key} must use YYYY-MM-DD") from exc
    if parsed.isoformat() != value:
        raise ValueError(f"{key} must use YYYY-MM-DD")
    return parsed


def evaluate(record):
    """Calculate the next due date from an explicit reporting interval and reference date."""
    previous = iso_date(record, "previous_report")
    as_of = iso_date(record, "as_of")
    interval = integer(record, "interval_days", minimum=1, maximum=365)
    if as_of < previous:
        raise ValueError("as_of must be on or after previous_report")
    due = previous + timedelta(days=interval)
    return {"due_on": due.isoformat(), "due": as_of >= due,
            "days_overdue": max(0, (as_of - due).days)}


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
    return {"asset": label(record, "asset"), "result": evaluate(record)}, 200


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

@app.get("/evaluate")
def show_help():
    return {"method": "POST", "content_type": "application/json", "project": PROJECT_ID}
```

## Implementation explanation

`evaluate()` applies the scheduled reporting rules to one record. `process()` implements usability improvement. The route handles media type, parsing, and status codes.

## Sample input data

Save this synthetic request as `sample.json`.

```json
{
  "asset": "LAB-17",
  "previous_report": "2026-06-01",
  "as_of": "2026-06-16",
  "interval_days": 7
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
  "days_overdue": 8
}
```

## Testing and validation

Actual validation used the FastAPI test client and synthetic requests. The sample returned HTTP 200 twice with identical JSON. The test observed the field below. An incomplete record returned HTTP 400. The help route returned the expected HTTP method. Malformed JSON and non-finite values returned HTTP 400 and the wrong media type returned HTTP 415.

```json
{
  "days_overdue": 8
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

Practice scheduled reporting using FastAPI and check a repeatable usability improvement workflow in the school lab scenario.

## Conclusion

The project records an observable scheduled reporting result for training case `LAB-17`.

[Back to the FastAPI project index](../README.md)

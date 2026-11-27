# Flask: Scheduled Reporting — Automation for Offline Classroom (Project 048477)

## Project overview

A classroom operating offline provides the setting for this scheduled reporting exercise. The embedded code is an AI-generated reference implementation based on the student guide and uses fictional data.

## Problem statement

The offline classroom needs a consistent HTTP interface for scheduled reporting and automation.

## Project objectives

- Calculate the next due date from an explicit reporting interval and reference date.
- Evaluate a batch while keeping each valid result and each row error.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Python, Flask, JSON, local HTTP, and the Python standard library.

## System architecture

`JSON POST → request validation → workflow → HTTP JSON response`

## Features

- Calculate the next due date from an explicit reporting interval and reference date.
- Evaluate a batch while keeping each valid result and each row error.

## Prerequisites

Install Python 3.10 or later and use curl for the example request. Use a text editor and a local practice directory.

## Complete Flask source code

```python
"""Local Flask reference implementation for project 048477."""

from datetime import date
from datetime import timedelta
import json
import math

from flask import Flask, jsonify, request

app = Flask(__name__)


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
def evaluate_route():
    if request.mimetype != "application/json":
        return jsonify(error="Send application/json"), 415
    try:
        payload = parse_json(request.get_data())
        body, status = process(payload)
    except (ValueError, OverflowError, RecursionError) as exc:
        return jsonify(error=str(exc)), 400
    response = jsonify(body)
    return response, status
```

## Implementation explanation

`evaluate()` applies the scheduled reporting rules to one record. `process()` implements automation. The route handles media type, parsing, and status codes.

## Sample input data

Save this synthetic request as `sample.json`.

```json
{
  "records": [
    {
      "asset": "CLASS-09",
      "previous_report": "2026-06-01",
      "as_of": "2026-06-16",
      "interval_days": 7
    },
    {
      "asset": "CLASS-09-B",
      "previous_report": "2026-06-01",
      "as_of": "2026-06-16",
      "interval_days": 7
    }
  ]
}
```

## Execution instructions

Save the code as `app.py` and the sample as `sample.json`. In a local virtual environment, install the dependency and start the server.

```sh
python -m pip install flask
python -m flask --app app run --host 127.0.0.1 --port 8000
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

Actual validation used the Flask test client and synthetic requests. The sample returned HTTP 200 twice with identical JSON. The test observed the field below. A mixed batch kept its valid row and reported one invalid row. Malformed JSON and non-finite values returned HTTP 400 and the wrong media type returned HTTP 415.

```json
{
  "days_overdue": 8
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

Practice scheduled reporting using Flask and check a repeatable automation workflow in the offline classroom scenario.

## Conclusion

The project records an observable scheduled reporting result for training case `CLASS-09`.

[Back to the Flask project index](../README.md)

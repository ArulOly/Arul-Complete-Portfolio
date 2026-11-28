# Flask: Monitoring — Reporting for Home Development (Project 048064)

## Project overview

A home development workspace provides the setting for this monitoring exercise. This exercise is based on the student guide and uses fictional data. This project includes source code, sample data, and instructions to run it.

## Problem statement

The home development needs a consistent HTTP interface for monitoring and reporting.

## Project objectives

- Summarize utilization readings and count readings above the limit.
- Return the result as a downloadable JSON attachment.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Python, Flask, JSON, local HTTP, and the Python standard library.

## System architecture

`JSON POST → request validation → workflow → HTTP JSON response`

## Features

- Summarize utilization readings and count readings above the limit.
- Return the result as a downloadable JSON attachment.

## Prerequisites

Install Python 3.10 or later and use curl for the example request. Use a text editor and a local practice directory.

## Complete Flask source code

```python
"""Local Flask reference implementation for project 048064."""

import json
import math
import statistics

from flask import Flask, jsonify, request

app = Flask(__name__)

PROJECT_ID = "048064"


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
def evaluate_route():
    if request.mimetype != "application/json":
        return jsonify(error="Send application/json"), 415
    try:
        payload = parse_json(request.get_data())
        body, status = process(payload)
    except (ValueError, OverflowError, RecursionError) as exc:
        return jsonify(error=str(exc)), 400
    response = jsonify(body)
    response.headers["Content-Disposition"] = f"attachment; filename=report-{PROJECT_ID}.json"
    return response, status
```

## Implementation explanation

`evaluate()` applies the monitoring rules to one record. `process()` implements reporting. The route handles media type, parsing, and status codes.

## Sample input data

Save this synthetic request as `sample.json`.

```json
{
  "asset": "DEV-05",
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
  "above_limit": 1
}
```

## Testing and validation

Actual validation used the Flask test client and synthetic requests. The sample returned HTTP 200 twice with identical JSON. The test observed the field below. An incomplete record returned HTTP 400. The response included the project-specific attachment filename. Malformed JSON and non-finite values returned HTTP 400 and the wrong media type returned HTTP 415.

```json
{
  "above_limit": 1
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

Practice monitoring using Flask and check a repeatable reporting workflow in the home development scenario.

## Conclusion

The project records an observable monitoring result for training case `DEV-05`.

[Back to the Flask project index](../README.md)

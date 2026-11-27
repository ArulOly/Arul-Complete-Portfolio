# Flask: Alerting — Usability Improvement for Offline Classroom (Project 048430)

## Project overview

A classroom operating offline provides the setting for this alerting exercise. The embedded code is an AI-generated reference implementation based on the student guide and uses fictional data.

## Problem statement

The offline classroom needs a consistent HTTP interface for alerting and usability improvement.

## Project objectives

- Emit one local alert per consecutive breach episode after a configured threshold.
- Offer a concise response and a GET route that explains the request format.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Python, Flask, JSON, local HTTP, and the Python standard library.

## System architecture

`JSON POST → request validation → workflow → HTTP JSON response`

## Features

- Emit one local alert per consecutive breach episode after a configured threshold.
- Offer a concise response and a GET route that explains the request format.

## Prerequisites

Install Python 3.10 or later and use curl for the example request. Use a text editor and a local practice directory.

## Complete Flask source code

```python
"""Local Flask reference implementation for project 048430."""

import json
import math

from flask import Flask, jsonify, request

app = Flask(__name__)

PROJECT_ID = "048430"


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
    """Emit one local alert per consecutive breach episode after a configured threshold."""
    limit = integer(record, "limit")
    needed = integer(record, "consecutive", minimum=1)
    streak = 0
    alerts = []
    for index, value in enumerate(series(record, "readings"), 1):
        streak = streak + 1 if value > limit else 0
        if streak == needed:
            alerts.append({"sample": index, "reading": value})
    return {"limit": limit, "alerts": alerts}


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

@app.get("/evaluate")
def show_help():
    return jsonify(method="POST", content_type="application/json", project=PROJECT_ID)
```

## Implementation explanation

`evaluate()` applies the alerting rules to one record. `process()` implements usability improvement. The route handles media type, parsing, and status codes.

## Sample input data

Save this synthetic request as `sample.json`.

```json
{
  "asset": "CLASS-09",
  "readings": [
    10,
    22,
    24,
    25,
    8,
    26,
    29
  ],
  "limit": 20,
  "consecutive": 2
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
  "alerts": [
    {
      "sample": 3,
      "reading": 24
    },
    {
      "sample": 7,
      "reading": 29
    }
  ]
}
```

## Testing and validation

Actual validation used the Flask test client and synthetic requests. The sample returned HTTP 200 twice with identical JSON. The test observed the field below. An incomplete record returned HTTP 400. The help route returned the expected HTTP method. Malformed JSON and non-finite values returned HTTP 400 and the wrong media type returned HTTP 415.

```json
{
  "alerts": [
    {
      "reading": 24,
      "sample": 3
    },
    {
      "reading": 29,
      "sample": 7
    }
  ]
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

Practice alerting using Flask and check a repeatable usability improvement workflow in the offline classroom scenario.

## Conclusion

The project records an observable alerting result for training case `CLASS-09`.

[Back to the Flask project index](../README.md)

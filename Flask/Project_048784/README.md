# Flask: Data Validation — Reporting for Single-Host Lab (Project 048784)

## Project overview

One simulated lab host provides the setting for this data validation exercise. The embedded code is an AI-generated reference implementation based on the student guide and uses fictional data.

## Problem statement

The single-host lab needs a consistent HTTP interface for data validation and reporting.

## Project objectives

- Return accepted values and row-level diagnostics for invalid or duplicate codes.
- Return the result as a downloadable JSON attachment.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Python, Flask, JSON, local HTTP, and the Python standard library.

## System architecture

`JSON POST → request validation → workflow → HTTP JSON response`

## Features

- Return accepted values and row-level diagnostics for invalid or duplicate codes.
- Return the result as a downloadable JSON attachment.

## Prerequisites

Install Python 3.10 or later and use curl for the example request. Use a text editor and a local practice directory.

## Complete Flask source code

```python
"""Local Flask reference implementation for project 048784."""

import json
import math

from flask import Flask, jsonify, request

app = Flask(__name__)

PROJECT_ID = "048784"


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
    """Return accepted values and row-level diagnostics for invalid or duplicate codes."""
    accepted = {}
    rejected = []
    for position, row in enumerate(objects(record, "rows"), 1):
        try:
            code = label(row, "code").upper()
            count = integer(row, "count")
            if code in accepted:
                raise ValueError("duplicate code " + code)
            accepted[code] = count
        except ValueError as exc:
            rejected.append({"row": position, "error": str(exc)})
    return {"accepted": accepted, "rejected": rejected}


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

`evaluate()` applies the data validation rules to one record. `process()` implements reporting. The route handles media type, parsing, and status codes.

## Sample input data

Save this synthetic request as `sample.json`.

```json
{
  "asset": "HOST-03",
  "rows": [
    {
      "code": "KIT-01",
      "count": 4
    },
    {
      "code": "KIT-01",
      "count": 2
    },
    {
      "code": "KIT-02",
      "count": -1
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
  "accepted": {
    "KIT-01": 4
  }
}
```

## Testing and validation

Actual validation used the Flask test client and synthetic requests. The sample returned HTTP 200 twice with identical JSON. The test observed the field below. An incomplete record returned HTTP 400. The response included the project-specific attachment filename. Malformed JSON and non-finite values returned HTTP 400 and the wrong media type returned HTTP 415.

```json
{
  "accepted": {
    "KIT-01": 4
  }
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

Practice data validation using Flask and check a repeatable reporting workflow in the single-host lab scenario.

## Conclusion

The project records an observable data validation result for training case `HOST-03`.

[Back to the Flask project index](../README.md)

# Flask: Data Import — Reporting for Single-Host Lab (Project 048284)

## Project overview

One simulated lab host provides the setting for this data import exercise. The embedded code is a reference implementation based on the student guide and uses fictional data.

## Problem statement

The single-host lab needs a consistent HTTP interface for data import and reporting.

## Project objectives

- Parse a small CSV with a fixed header and turn rows into normalized records.
- Return the result as a downloadable JSON attachment.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Python, Flask, JSON, local HTTP, and the Python standard library.

## System architecture

`JSON POST → request validation → workflow → HTTP JSON response`

## Features

- Parse a small CSV with a fixed header and turn rows into normalized records.
- Return the result as a downloadable JSON attachment.

## Prerequisites

Install Python 3.10 or later and use curl for the example request. Use a text editor and a local practice directory.

## Complete Flask source code

```python
"""Local Flask reference implementation for project 048284."""

import csv
import io
import json
import math

from flask import Flask, jsonify, request

app = Flask(__name__)

PROJECT_ID = "048284"


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


def evaluate(record):
    """Parse a small CSV with a fixed header and turn rows into normalized records."""
    reader = csv.DictReader(io.StringIO(label(record, "csv_data")), strict=True)
    if reader.fieldnames != ["code", "quantity"]:
        raise ValueError("CSV must begin with code,quantity")
    rows = []
    seen = set()
    try:
        for row in reader:
            if None in row or row["quantity"] is None:
                raise ValueError("each CSV row must contain exactly two fields")
            code = label(row, "code")
            quantity = integer({"quantity": int(row["quantity"])}, "quantity")
            if code in seen:
                raise ValueError("duplicate CSV code: " + code)
            seen.add(code)
            rows.append({"code": code, "quantity": quantity})
    except csv.Error as exc:
        raise ValueError(f"invalid CSV: {exc}") from exc
    if not rows:
        raise ValueError("CSV must contain at least one data row")
    return {"imported": len(rows), "rows": rows}


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

`evaluate()` applies the data import rules to one record. `process()` implements reporting. The route handles media type, parsing, and status codes.

## Sample input data

Save this synthetic request as `sample.json`.

```json
{
  "asset": "HOST-03",
  "csv_data": "code,quantity\nKIT-01,4\nKIT-02,2\n"
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
  "imported": 2
}
```

## Testing and validation

Actual validation used the Flask test client and synthetic requests. The sample returned HTTP 200 twice with identical JSON. The test observed the field below. An incomplete record returned HTTP 400. The response included the project-specific attachment filename. Malformed JSON and non-finite values returned HTTP 400 and the wrong media type returned HTTP 415.

```json
{
  "imported": 2
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

Practice data import using Flask and check a repeatable reporting workflow in the single-host lab scenario.

## Conclusion

The project records an observable data import result for training case `HOST-03`.

[Back to the Flask project index](../README.md)

# Flask: Configuration Comparison — Reporting for Public Dataset (Project 048669)

## Project overview

A synthetic public-data exercise provides the setting for this configuration comparison exercise. This exercise is based on the student guide and uses fictional data. This project includes source code, sample data, and instructions to run it.

## Problem statement

The public dataset needs a consistent HTTP interface for configuration comparison and reporting.

## Project objectives

- Identify added, removed, and changed settings between two configuration objects.
- Return the result as a downloadable JSON attachment.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Python, Flask, JSON, local HTTP, and the Python standard library.

## System architecture

`JSON POST → request validation → workflow → HTTP JSON response`

## Features

- Identify added, removed, and changed settings between two configuration objects.
- Return the result as a downloadable JSON attachment.

## Prerequisites

Install Python 3.10 or later and use curl for the example request. Use a text editor and a local practice directory.

## Complete Flask source code

```python
"""Local Flask reference implementation for project 048669."""

import json
import math

from flask import Flask, jsonify, request

app = Flask(__name__)

PROJECT_ID = "048669"


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

`evaluate()` applies the configuration comparison rules to one record. `process()` implements reporting. The route handles media type, parsing, and status codes.

## Sample input data

Save this synthetic request as `sample.json`.

```json
{
  "asset": "DATA-12",
  "baseline": {
    "timeout": 30,
    "theme": "light"
  },
  "current": {
    "timeout": 45,
    "retries": 2
  }
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
  "changed": [
    "timeout"
  ]
}
```

## Testing and validation

Actual validation used the Flask test client and synthetic requests. The sample returned HTTP 200 twice with identical JSON. The test observed the field below. An incomplete record returned HTTP 400. The response included the project-specific attachment filename. Malformed JSON and non-finite values returned HTTP 400 and the wrong media type returned HTTP 415.

```json
{
  "changed": [
    "timeout"
  ]
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

Practice configuration comparison using Flask and check a repeatable reporting workflow in the public dataset scenario.

## Conclusion

The project records an observable configuration comparison result for training case `DATA-12`.

[Back to the Flask project index](../README.md)

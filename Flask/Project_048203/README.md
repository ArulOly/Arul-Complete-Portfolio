# Flask: Backup Verification — Recovery for School Lab (Project 048203)

## Project overview

Shared classroom equipment provides the setting for this backup verification exercise. The embedded code is a reference implementation based on the student guide and uses fictional data.

## Problem statement

The school lab needs a consistent HTTP interface for backup verification and recovery.

## Project objectives

- Compare a synthetic backup payload against an expected SHA-256 digest.
- Evaluate a fallback record when the primary input fails validation.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Python, Flask, JSON, local HTTP, and the Python standard library.

## System architecture

`JSON POST → request validation → workflow → HTTP JSON response`

## Features

- Compare a synthetic backup payload against an expected SHA-256 digest.
- Evaluate a fallback record when the primary input fails validation.

## Prerequisites

Install Python 3.10 or later and use curl for the example request. Use a text editor and a local practice directory.

## Complete Flask source code

```python
"""Local Flask reference implementation for project 048203."""

import hashlib
import json
import math

from flask import Flask, jsonify, request

app = Flask(__name__)


def label(record, key):
    value = record.get(key)
    if not isinstance(value, str) or not value.strip():
        raise ValueError(f"{key} must be non-empty text")
    return value.strip()


def evaluate(record):
    """Compare a synthetic backup payload against an expected SHA-256 digest."""
    expected = label(record, "sha256").lower()
    if len(expected) != 64 or any(char not in "0123456789abcdef" for char in expected):
        raise ValueError("sha256 must contain exactly 64 hexadecimal characters")
    actual = hashlib.sha256(label(record, "payload").encode("utf-8")).hexdigest()
    return {"bytes": len(label(record, "payload").encode("utf-8")),
            "sha256": actual, "verified": actual == expected}


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
    envelope = as_object(payload)
    try:
        record = as_object(envelope.get("primary"))
        return {"source": "primary", "result": evaluate(record)}, 200
    except (ValueError, OverflowError) as exc:
        record = as_object(envelope.get("fallback"))
        return {"source": "fallback", "primary_error": str(exc), "result": evaluate(record)}, 200


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

`evaluate()` applies the backup verification rules to one record. `process()` implements recovery. The route handles media type, parsing, and status codes.

## Sample input data

Save this synthetic request as `sample.json`.

```json
{
  "primary": {
    "asset": "LAB-17",
    "payload": "Backup manifest for LAB-17, revision 1",
    "sha256": "86df86425b2c40036fd7f1ae47c08f4cb03b768bfe350d021d7a49cb1c0121c6"
  },
  "fallback": {
    "asset": "LAB-17",
    "payload": "Backup manifest for LAB-17, revision 1",
    "sha256": "86df86425b2c40036fd7f1ae47c08f4cb03b768bfe350d021d7a49cb1c0121c6"
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
  "verified": true
}
```

## Testing and validation

Actual validation used the Flask test client and synthetic requests. The sample returned HTTP 200 twice with identical JSON. The test observed the field below. An incomplete primary used the fallback, and invalid primary and fallback records returned HTTP 400. Malformed JSON and non-finite values returned HTTP 400 and the wrong media type returned HTTP 415.

```json
{
  "verified": true
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

Practice backup verification using Flask and check a repeatable recovery workflow in the school lab scenario.

## Conclusion

The project records an observable backup verification result for training case `LAB-17`.

[Back to the Flask project index](../README.md)

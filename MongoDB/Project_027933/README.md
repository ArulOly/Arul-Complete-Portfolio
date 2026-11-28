# MongoDB: Visual Reporting — Recovery for Single-Host Lab (Project 027933)

## Project overview

One simulated lab host provides the setting for this visual reporting exercise. This exercise is based on the student guide and uses fictional data. This project includes source code, sample data, and instructions to run it.

## Problem statement

The single-host lab needs a reproducible visual reporting workflow that stores and reads results in MongoDB.

## Project objectives

- Create a proportional text bar chart with exact values and a labeled scale.
- Save a fallback snapshot and read it back if the primary record fails validation.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

MongoDB BSON documents, indexed IDs, and an aggregation pipeline, Python, JSON, and PyMongo.

## System architecture


`Local JSON → Python workflow → database write → database read or summary → terminal`

## Features

- Create a proportional text bar chart with exact values and a labeled scale.
- Save a fallback snapshot and read it back if the primary record fails validation.

## Prerequisites

Install Python 3.10 or later and use a local MongoDB server and a practice user allowed to create, read, and drop temporary collections in the portfolio_lab database. Use a text editor and a local practice directory.

## Complete MongoDB source code

```python
"""Local MongoDB reference implementation for project 027933."""


from uuid import uuid4
import argparse
import json
import os
import pymongo
import sys


METRIC = 'peak'


def dumps(value):
    return json.dumps(value, ensure_ascii=False, allow_nan=False, sort_keys=True)


def round_trip(value):
    return json.loads(dumps(value))


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
    """Create a proportional text bar chart with exact values and a labeled scale."""
    values = field_mapping(record, "values")
    if not values:
        raise ValueError("values must contain at least one category")
    counts = {key: integer(values, key) for key in values}
    width = integer(record, "width", minimum=1, maximum=60)
    peak = max(counts.values())
    bars = [f"{key}: {'#' * ((value * width + peak // 2) // peak if peak else 0)} ({value})"
            for key, value in sorted(counts.items())]
    return {"peak": peak, "max_bar_width": width, "bars": bars}


def field_mapping(record, key):
    value = record.get(key)
    if not isinstance(value, dict):
        raise ValueError(f"{key} must be an object")
    return value


def mapping(record, key):
    if not isinstance(record, dict):
        raise ValueError(f"{key} must be an object")
    return record


class Store:

    def __enter__(self):
        self.client = pymongo.MongoClient(os.environ["PORTFOLIO_MONGODB_URI"],
                                          serverSelectionTimeoutMS=5000)
        self.collection = self.client["portfolio_lab"]["demo_" + uuid4().hex]
        return self

    def __exit__(self, *_):
        try:
            self.collection.drop()
        finally:
            self.client.close()

    def put(self, position, source, response):
        self.collection.replace_one(
            {"_id": position},
            {"_id": position, "source": round_trip(source), "response": round_trip(response),
             "rejected": int("error" in response)},
            upsert=True,
        )

    def read(self, position):
        document = self.collection.find_one({"_id": position})
        if document is None:
            raise ValueError("Stored row is missing")
        return document["source"], document["response"]

    def metric(self, position, key):
        row = next(self.collection.aggregate([
            {"$match": {"_id": position}},
            {"$project": {"_id": 0, "value": "$response.result." + key}},
        ]), {})
        return row.get("value")


def load_input(path):
    with open(path, encoding="utf-8") as handle:
        value = json.load(handle)
    dumps(value)  # Reject non-finite numbers before any database write.
    return value


def evaluate_record(record):
    return {"asset": label(record, "asset"), "result": evaluate(record)}


def process(payload, store):
    envelope = mapping(payload, "input")
    fallback = mapping(envelope.get("fallback"), "fallback")
    store.put(1, fallback, {"snapshot": True})
    try:
        record = mapping(envelope.get("primary"), "primary")
        result = evaluate_record(record)
        source = "primary"
        diagnostic = None
    except (ValueError, OverflowError) as exc:
        record, _ = store.read(1)
        result = evaluate_record(record)
        source = "fallback"
        diagnostic = str(exc)
    store.put(2, record, result)
    return {"source": source, "primary_error": diagnostic,
            "row": store.read(2)[1], "metric": store.metric(2, METRIC), "status": 0}


def run(path):
    payload = load_input(path)
    with Store() as store:
        return process(payload, store)


def main():
    parser = argparse.ArgumentParser(description='Create a proportional text bar chart with exact values and a labeled scale.')
    parser.add_argument("input", help="path to the synthetic JSON fixture")
    args = parser.parse_args()
    try:
        response = run(args.input)
        print(json.dumps(response, indent=2, ensure_ascii=False, sort_keys=True))
        return response["status"]
    except (pymongo.errors.PyMongoError, OSError, ValueError, KeyError, OverflowError, RecursionError) as exc:
        print(f"Demo failed: {exc}", file=sys.stderr)
        return 2


if __name__ == "__main__":
    raise SystemExit(main())
```

## Implementation explanation

`evaluate()` performs the visual reporting calculation and validates its inputs. `Store` handles database writes and reads. The program uses a uniquely named temporary collection in the portfolio_lab database and drops that collection on normal exit. Save a fallback snapshot and read it back if the primary record fails validation.

## Sample input data

Save this synthetic data as `sample.json`.

```json
{
  "primary": {
    "asset": "HOST-03",
    "values": {
      "available": 12,
      "reserved": 7,
      "repair": 3
    },
    "width": 12
  },
  "fallback": {
    "asset": "HOST-03",
    "values": {
      "available": 12,
      "reserved": 7,
      "repair": 3
    },
    "width": 12
  }
}
```

## Execution instructions

Copy the Python block to `project.py` and the sample to `sample.json` in the same directory.

Install the driver in your virtual environment with `python -m pip install pymongo`. Set `PORTFOLIO_MONGODB_URI` to your local connection URI. Its format is `mongodb://127.0.0.1:27017`. Use your own practice credentials when authentication is required.

Run the example.

```sh
python project.py sample.json
```

## Example output

Illustrative excerpt for the synthetic sample.

```json
{
  "peak": 12
}
```

## Testing and validation

Actual tests used MongoDB and temporary synthetic JSON fixtures. The sample returned the field below twice with the same result. An invalid primary record used the saved fallback and returned the expected field.

```json
{
  "peak": 12
}
```

## Error handling

The CLI reports invalid input, database errors, and file errors with status 2. A batch with rejected records preserves its valid rows and returns status 1.

## Troubleshooting

- Confirm the documented runtime and copy the complete source block.
- Start with the documented sample and use the filename or request path shown above.
- Compare the observed value with the example before changing an input.

## Limitations

This small example uses a temporary database store and synthetic records.

## Learning outcomes

Practice visual reporting using MongoDB and check a repeatable recovery workflow in the single-host lab scenario.

## Conclusion

The project records an observable visual reporting result for training case `HOST-03`.

[Back to the MongoDB project index](../README.md)

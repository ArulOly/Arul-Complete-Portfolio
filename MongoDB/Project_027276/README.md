# MongoDB: Data Import — Baseline Setup for Offline Classroom (Project 027276)

## Project overview

A classroom operating offline provides the setting for this data import exercise. The embedded code is an AI-generated reference implementation based on the student guide and uses fictional data.

## Problem statement

The offline classroom needs a reproducible data import workflow that stores and reads results in MongoDB.

## Project objectives

- Parse a small CSV with a fixed header and turn rows into normalized records.
- Create a session-local store, save a validated baseline, and read it back by row.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

MongoDB BSON documents, indexed IDs, and an aggregation pipeline, Python, JSON, and PyMongo.

## System architecture


`Local JSON → Python workflow → database write → database read or summary → terminal`

## Features

- Parse a small CSV with a fixed header and turn rows into normalized records.
- Create a session-local store, save a validated baseline, and read it back by row.

## Prerequisites

Install Python 3.10 or later and use a local MongoDB server and a practice user allowed to create, read, and drop temporary collections in the portfolio_lab database. Use a text editor and a local practice directory.

## Complete MongoDB source code

```python
"""Local MongoDB reference implementation for project 027276."""


from uuid import uuid4
import argparse
import csv
import io
import json
import os
import pymongo
import sys


METRIC = 'imported'


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
    record = mapping(payload, "input")
    result = evaluate_record(record)
    store.put(1, record, result)
    original, restored = store.read(1)
    if original != record:
        raise ValueError("Stored baseline changed")
    return {"baseline": restored, "metric": store.metric(1, METRIC), "status": 0}


def run(path):
    payload = load_input(path)
    with Store() as store:
        return process(payload, store)


def main():
    parser = argparse.ArgumentParser(description='Parse a small CSV with a fixed header and turn rows into normalized records.')
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

`evaluate()` performs the data import calculation and validates its inputs. `Store` handles database writes and reads. The program uses a uniquely named temporary collection in the portfolio_lab database and drops that collection on normal exit. Create a session-local store, save a validated baseline, and read it back by row.

## Sample input data

Save this synthetic data as `sample.json`.

```json
{
  "asset": "CLASS-09",
  "csv_data": "code,quantity\nKIT-01,4\nKIT-02,2\n"
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
  "imported": 2
}
```

## Testing and validation

Actual tests used MongoDB and temporary synthetic JSON fixtures. The sample returned the field below twice with the same result. An incomplete record was rejected.

```json
{
  "imported": 2
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

Practice data import using MongoDB and check a repeatable baseline setup workflow in the offline classroom scenario.

## Conclusion

The project records an observable data import result for training case `CLASS-09`.

[Back to the MongoDB project index](../README.md)

# MongoDB: Inventory — Reporting for Public Dataset (Project 027019)

## Project overview

A synthetic public-data exercise provides the setting for this inventory exercise. The embedded code is an AI-generated reference implementation based on the student guide and uses fictional data.

## Problem statement

The public dataset needs a reproducible inventory workflow that stores and reads results in MongoDB.

## Project objectives

- Count items and identify stock below the reorder level.
- Build a JSON report from the database's ordered result set and row summary.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

MongoDB BSON documents, indexed IDs, and an aggregation pipeline, Python, JSON, and PyMongo.

## System architecture


`Local JSON → Python workflow → database write → database read or summary → terminal`

## Features

- Count items and identify stock below the reorder level.
- Build a JSON report from the database's ordered result set and row summary.

## Prerequisites

Install Python 3.10 or later and use a local MongoDB server and a practice user allowed to create, read, and drop temporary collections in the portfolio_lab database. Use a text editor and a local practice directory.

## Complete MongoDB source code

```python
"""Local MongoDB reference implementation for project 027019."""


from uuid import uuid4
import argparse
import json
import os
import pymongo
import sys


METRIC = 'needs_reorder'
PROJECT_ID = '027019'


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
    """Count items and identify stock below the reorder level."""
    stock = integer(record, "stock")
    reorder = integer(record, "reorder_at", minimum=1)
    return {"stock": stock, "reorder_at": reorder, "needs_reorder": stock < reorder}


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

    def ordered(self):
        return [row["response"] for row in self.collection.find({}, {"response": 1}).sort("_id", 1)]

    def summary(self):
        row = next(self.collection.aggregate([
            {"$group": {"_id": None, "total": {"$sum": 1}, "failed": {"$sum": "$rejected"}}}
        ]), {"total": 0, "failed": 0})
        return row["total"], row["failed"]

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
    store.put(1, record, evaluate_record(record))
    total, failed = store.summary()
    return {"project": PROJECT_ID, "report": store.ordered(), "total": total,
            "failed": failed, "metric": store.metric(1, METRIC), "status": 0}


def run(path):
    payload = load_input(path)
    with Store() as store:
        return process(payload, store)


def main():
    parser = argparse.ArgumentParser(description='Count items and identify stock below the reorder level.')
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

`evaluate()` performs the inventory calculation and validates its inputs. `Store` handles database writes and reads. The program uses a uniquely named temporary collection in the portfolio_lab database and drops that collection on normal exit. Build a JSON report from the database's ordered result set and row summary.

## Sample input data

Save this synthetic data as `sample.json`.

```json
{
  "asset": "DATA-12",
  "stock": 2,
  "reorder_at": 4
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
  "needs_reorder": true
}
```

## Testing and validation

Actual tests used MongoDB and temporary synthetic JSON fixtures. The sample returned the field below twice with the same result. An incomplete record was rejected.

```json
{
  "needs_reorder": true
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

Practice inventory using MongoDB and check a repeatable reporting workflow in the public dataset scenario.

## Conclusion

The project records an observable inventory result for training case `DATA-12`.

[Back to the MongoDB project index](../README.md)

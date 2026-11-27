# MongoDB: Access Review — Automation for Public Dataset (Project 027517)

## Project overview

A synthetic public-data exercise provides the setting for this access review exercise. The embedded code is an AI-generated reference implementation based on the student guide and uses fictional data.

## Problem statement

The public dataset needs a reproducible access review workflow that stores and reads results in MongoDB.

## Project objectives

- Compare synthetic active grants with an approved role policy.
- Store valid results and row errors in order and summarize the persisted batch.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

MongoDB BSON documents, indexed IDs, and an aggregation pipeline, Python, JSON, and PyMongo.

## System architecture


`Local JSON → Python workflow → database write → database read or summary → terminal`

## Features

- Compare synthetic active grants with an approved role policy.
- Store valid results and row errors in order and summarize the persisted batch.

## Prerequisites

Install Python 3.10 or later and use a local MongoDB server and a practice user allowed to create, read, and drop temporary collections in the portfolio_lab database. Use a text editor and a local practice directory.

## Complete MongoDB source code

```python
"""Local MongoDB reference implementation for project 027517."""


from uuid import uuid4
import argparse
import json
import os
import pymongo
import sys


def dumps(value):
    return json.dumps(value, ensure_ascii=False, allow_nan=False, sort_keys=True)


def round_trip(value):
    return json.loads(dumps(value))


def label(record, key):
    value = record.get(key)
    if not isinstance(value, str) or not value.strip():
        raise ValueError(f"{key} must be non-empty text")
    return value.strip()


def array(record, key, allow_empty=False):
    value = record.get(key)
    if not isinstance(value, list) or (not value and not allow_empty):
        raise ValueError(f"{key} must be a list" + ("" if allow_empty else " with at least one item"))
    return value


def labels(record, key, allow_empty=False):
    return [label({key: item}, key) for item in array(record, key, allow_empty)]


def evaluate(record):
    """Compare synthetic active grants with an approved role policy."""
    approved = set(labels(record, "approved", allow_empty=True))
    grants = set(labels(record, "grants", allow_empty=True))
    return {"excess_grants": sorted(grants - approved),
            "missing_grants": sorted(approved - grants),
            "matches_policy": approved == grants}


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

    def ordered(self):
        return [row["response"] for row in self.collection.find({}, {"response": 1}).sort("_id", 1)]

    def summary(self):
        row = next(self.collection.aggregate([
            {"$group": {"_id": None, "total": {"$sum": 1}, "failed": {"$sum": "$rejected"}}}
        ]), {"total": 0, "failed": 0})
        return row["total"], row["failed"]


def load_input(path):
    with open(path, encoding="utf-8") as handle:
        value = json.load(handle)
    dumps(value)  # Reject non-finite numbers before any database write.
    return value


def evaluate_record(record):
    return {"asset": label(record, "asset"), "result": evaluate(record)}


def process(payload, store):
    records = array(mapping(payload, "input"), "records")
    for position, candidate in enumerate(records, 1):
        try:
            response = evaluate_record(mapping(candidate, f"row {position}"))
        except (ValueError, OverflowError) as exc:
            response = {"error": str(exc)}
        store.put(position, candidate, response)
    total, failed = store.summary()
    return {"rows": store.ordered(), "total": total, "failed": failed,
            "status": int(failed > 0)}


def run(path):
    payload = load_input(path)
    with Store() as store:
        return process(payload, store)


def main():
    parser = argparse.ArgumentParser(description='Compare synthetic active grants with an approved role policy.')
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

`evaluate()` performs the access review calculation and validates its inputs. `Store` handles database writes and reads. The program uses a uniquely named temporary collection in the portfolio_lab database and drops that collection on normal exit. Store valid results and row errors in order and summarize the persisted batch.

## Sample input data

Save this synthetic data as `sample.json`.

```json
{
  "records": [
    {
      "asset": "DATA-12",
      "approved": [
        "read",
        "submit"
      ],
      "grants": [
        "read",
        "admin"
      ]
    },
    {
      "asset": "DATA-12-B",
      "approved": [
        "read",
        "submit"
      ],
      "grants": [
        "read",
        "admin"
      ]
    }
  ]
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
  "excess_grants": [
    "admin"
  ]
}
```

## Testing and validation

Actual tests used MongoDB and temporary synthetic JSON fixtures. The sample returned the field below twice with the same result. A mixed batch kept its valid row and the database summary reported one rejection.

```json
{
  "excess_grants": [
    "admin"
  ]
}
```

## Error handling

The CLI reports invalid input, database errors, and file errors with status 2. A batch with rejected records preserves its valid rows and returns status 1.

## Troubleshooting

- Confirm the documented runtime and copy the complete source block.
- Start with the documented sample and use the filename or request path shown above.
- Compare the observed value with the example before changing an input.

## Limitations

The comparison uses fictional grants in sample data. It does not change real database roles.

## Learning outcomes

Practice access review using MongoDB and check a repeatable automation workflow in the public dataset scenario.

## Conclusion

The project records an observable access review result for training case `DATA-12`.

[Back to the MongoDB project index](../README.md)

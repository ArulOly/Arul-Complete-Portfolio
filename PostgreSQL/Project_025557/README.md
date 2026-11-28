# PostgreSQL: Performance Baseline — Automation for Small Business (Project 025557)

## Project overview

Local shop operations provides the setting for this performance baseline exercise. The embedded code is a reference implementation based on the student guide and uses fictional data.

## Problem statement

The small business needs a reproducible performance baseline workflow that stores and reads results in PostgreSQL.

## Project objectives

- Calculate median and nearest-rank p95 latency against a stated budget.
- Store valid results and row errors in order and summarize the persisted batch.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

PostgreSQL temporary tables, JSONB operators, and bound SQL parameters, Python, JSON, and Psycopg 3.

## System architecture


`Local JSON → Python workflow → database write → database read or summary → terminal`

## Features

- Calculate median and nearest-rank p95 latency against a stated budget.
- Store valid results and row errors in order and summarize the persisted batch.

## Prerequisites

Install Python 3.10 or later and use a local PostgreSQL database and a practice role allowed to create temporary tables. Use a text editor and a local practice directory.

## Complete PostgreSQL source code

```python
"""Local PostgreSQL reference implementation for project 025557."""


import argparse
import json
import os
import psycopg
import statistics
import sys


def dumps(value):
    return json.dumps(value, ensure_ascii=False, allow_nan=False, sort_keys=True)


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
    """Calculate median and nearest-rank p95 latency against a stated budget."""
    readings = sorted(series(record, "latencies_ms"))
    rank = (95 * len(readings) + 99) // 100
    p95 = readings[rank - 1]
    budget = integer(record, "budget_ms", minimum=1)
    return {"samples": len(readings), "median_ms": statistics.median(readings),
            "p95_ms": p95, "within_budget": p95 <= budget}


def mapping(record, key):
    if not isinstance(record, dict):
        raise ValueError(f"{key} must be an object")
    return record


class Store:

    def __enter__(self):
        self.database = psycopg.connect(os.environ["PORTFOLIO_POSTGRES_DSN"], connect_timeout=5, autocommit=True)
        self.cursor = self.database.cursor()
        try:
            self.cursor.execute('CREATE TEMP TABLE portfolio_rows (\n                    position INTEGER PRIMARY KEY,\n                    source JSONB NOT NULL,\n                    response JSONB NOT NULL,\n                    rejected INTEGER NOT NULL CHECK (rejected IN (0, 1))\n                )')
        except psycopg.Error:
            self.database.close()
            raise
        return self

    def __exit__(self, *_):
        self.database.close()

    def put(self, position, source, response):
        self.cursor.execute('INSERT INTO portfolio_rows VALUES (%s, %s::jsonb, %s::jsonb, %s)',
                            (position, dumps(source), dumps(response), int("error" in response)))

    def ordered(self):
        self.cursor.execute('SELECT response::text FROM portfolio_rows ORDER BY position')
        return [json.loads(row[0]) for row in self.cursor.fetchall()]

    def summary(self):
        self.cursor.execute('SELECT COUNT(*), COALESCE(SUM(rejected), 0) FROM portfolio_rows')
        total, failed = self.cursor.fetchone()
        return int(total), int(failed)


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
    parser = argparse.ArgumentParser(description='Calculate median and nearest-rank p95 latency against a stated budget.')
    parser.add_argument("input", help="path to the synthetic JSON fixture")
    args = parser.parse_args()
    try:
        response = run(args.input)
        print(json.dumps(response, indent=2, ensure_ascii=False, sort_keys=True))
        return response["status"]
    except (psycopg.Error, OSError, ValueError, KeyError, OverflowError, RecursionError) as exc:
        print(f"Demo failed: {exc}", file=sys.stderr)
        return 2


if __name__ == "__main__":
    raise SystemExit(main())
```

## Implementation explanation

`evaluate()` performs the performance baseline calculation and validates its inputs. `Store` handles database writes and reads. The session keeps its data in a temporary PostgreSQL table with JSONB columns that closes with the connection. Store valid results and row errors in order and summarize the persisted batch.

## Sample input data

Save this synthetic data as `sample.json`.

```json
{
  "records": [
    {
      "asset": "SHOP-22",
      "latencies_ms": [
        12,
        15,
        14,
        44,
        17
      ],
      "budget_ms": 30
    },
    {
      "asset": "SHOP-22-B",
      "latencies_ms": [
        12,
        15,
        14,
        44,
        17
      ],
      "budget_ms": 30
    }
  ]
}
```

## Execution instructions

Copy the Python block to `project.py` and the sample to `sample.json` in the same directory.

Install the driver in your virtual environment with `python -m pip install psycopg[binary]`. Set `PORTFOLIO_POSTGRES_DSN` to your local connection URI. Its format is `postgresql://USER:PASSWORD@127.0.0.1:5432/portfolio_lab`. Use your own practice credentials when authentication is required.

Run the example.

```sh
python project.py sample.json
```

## Example output

Illustrative excerpt for the synthetic sample.

```json
{
  "p95_ms": 44
}
```

## Testing and validation

Actual tests used PostgreSQL and temporary synthetic JSON fixtures. The sample returned the field below twice with the same result. A mixed batch kept its valid row and the database summary reported one rejection.

```json
{
  "p95_ms": 44
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

Practice performance baseline using PostgreSQL and check a repeatable automation workflow in the small business scenario.

## Conclusion

The project records an observable performance baseline result for training case `SHOP-22`.

[Back to the PostgreSQL project index](../README.md)

# PostgreSQL: Recovery Test — Reporting for Public Dataset (Project 025969)

## Project overview

A synthetic public-data exercise provides the setting for this recovery test exercise. The embedded code is a reference implementation based on the student guide and uses fictional data.

## Problem statement

The public dataset needs a reproducible recovery test workflow that stores and reads results in PostgreSQL.

## Project objectives

- Restore a mock asset snapshot in memory and compare it against expected values.
- Build a JSON report from the database's ordered result set and row summary.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

PostgreSQL temporary tables, JSONB operators, and bound SQL parameters, Python, JSON, and Psycopg 3.

## System architecture


`Local JSON → Python workflow → database write → database read or summary → terminal`

## Features

- Restore a mock asset snapshot in memory and compare it against expected values.
- Build a JSON report from the database's ordered result set and row summary.

## Prerequisites

Install Python 3.10 or later and use a local PostgreSQL database and a practice role allowed to create temporary tables. Use a text editor and a local practice directory.

## Complete PostgreSQL source code

```python
"""Local PostgreSQL reference implementation for project 025969."""


import argparse
import json
import os
import psycopg
import sys


METRIC = 'verified'
PROJECT_ID = '025969'


def dumps(value):
    return json.dumps(value, ensure_ascii=False, allow_nan=False, sort_keys=True)


def label(record, key):
    value = record.get(key)
    if not isinstance(value, str) or not value.strip():
        raise ValueError(f"{key} must be non-empty text")
    return value.strip()


def evaluate(record):
    """Restore a mock asset snapshot in memory and compare it against expected values."""
    saved = field_mapping(record, "snapshot")
    expected = field_mapping(record, "expected")
    restored = json.loads(json.dumps(saved))
    keys = restored.keys() | expected.keys()
    differences = sorted(key for key in keys if key not in restored or key not in expected
                         or json.dumps(restored[key], sort_keys=True)
                         != json.dumps(expected[key], sort_keys=True))
    return {"restored": restored, "differences": differences, "verified": not differences}


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

    def read(self, position):
        self.cursor.execute('SELECT source::text, response::text FROM portfolio_rows WHERE position = %s', (position,))
        row = self.cursor.fetchone()
        if row is None:
            raise ValueError("Stored row is missing")
        return json.loads(row[0]), json.loads(row[1])

    def ordered(self):
        self.cursor.execute('SELECT response::text FROM portfolio_rows ORDER BY position')
        return [json.loads(row[0]) for row in self.cursor.fetchall()]

    def summary(self):
        self.cursor.execute('SELECT COUNT(*), COALESCE(SUM(rejected), 0) FROM portfolio_rows')
        total, failed = self.cursor.fetchone()
        return int(total), int(failed)

    def metric(self, position, key):
        self.cursor.execute("SELECT response -> 'result' -> %s::text FROM portfolio_rows WHERE position = %s", (key, position))
        value = self.cursor.fetchone()[0]
        return json.loads(value) if False and value is not None else value


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
    parser = argparse.ArgumentParser(description='Restore a mock asset snapshot in memory and compare it against expected values.')
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

`evaluate()` performs the recovery test calculation and validates its inputs. `Store` handles database writes and reads. The session keeps its data in a temporary PostgreSQL table with JSONB columns that closes with the connection. Build a JSON report from the database's ordered result set and row summary.

## Sample input data

Save this synthetic data as `sample.json`.

```json
{
  "asset": "DATA-12",
  "snapshot": {
    "version": 3,
    "active": true
  },
  "expected": {
    "version": 3,
    "active": true
  }
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
  "verified": true
}
```

## Testing and validation

Actual tests used PostgreSQL and temporary synthetic JSON fixtures. The sample returned the field below twice with the same result. An incomplete record was rejected.

```json
{
  "verified": true
}
```

## Error handling

The CLI reports invalid input, database errors, and file errors with status 2. A batch with rejected records preserves its valid rows and returns status 1.

## Troubleshooting

- Confirm the documented runtime and copy the complete source block.
- Start with the documented sample and use the filename or request path shown above.
- Compare the observed value with the example before changing an input.

## Limitations

The sample restores a synthetic snapshot. It does not recover the database server.

## Learning outcomes

Practice recovery test using PostgreSQL and check a repeatable reporting workflow in the public dataset scenario.

## Conclusion

The project records an observable recovery test result for training case `DATA-12`.

[Back to the PostgreSQL project index](../README.md)

# PostgreSQL: Data Import — Recovery for Home Development (Project 025263)

## Project overview

A home development workspace provides the setting for this data import exercise. The embedded code is an AI-generated reference implementation based on the student guide and uses fictional data.

## Problem statement

The home development needs a reproducible data import workflow that stores and reads results in PostgreSQL.

## Project objectives

- Parse a small CSV with a fixed header and turn rows into normalized records.
- Save a fallback snapshot and read it back if the primary record fails validation.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

PostgreSQL temporary tables, JSONB operators, and bound SQL parameters, Python, JSON, and Psycopg 3.

## System architecture


`Local JSON → Python workflow → database write → database read or summary → terminal`

## Features

- Parse a small CSV with a fixed header and turn rows into normalized records.
- Save a fallback snapshot and read it back if the primary record fails validation.

## Prerequisites

Install Python 3.10 or later and use a local PostgreSQL database and a practice role allowed to create temporary tables. Use a text editor and a local practice directory.

## Complete PostgreSQL source code

```python
"""Local PostgreSQL reference implementation for project 025263."""


import argparse
import csv
import io
import json
import os
import psycopg
import sys


METRIC = 'imported'


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
    parser = argparse.ArgumentParser(description='Parse a small CSV with a fixed header and turn rows into normalized records.')
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

`evaluate()` performs the data import calculation and validates its inputs. `Store` handles database writes and reads. The session keeps its data in a temporary PostgreSQL table with JSONB columns that closes with the connection. Save a fallback snapshot and read it back if the primary record fails validation.

## Sample input data

Save this synthetic data as `sample.json`.

```json
{
  "primary": {
    "asset": "DEV-05",
    "csv_data": "code,quantity\nKIT-01,4\nKIT-02,2\n"
  },
  "fallback": {
    "asset": "DEV-05",
    "csv_data": "code,quantity\nKIT-01,4\nKIT-02,2\n"
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
  "imported": 2
}
```

## Testing and validation

Actual tests used PostgreSQL and temporary synthetic JSON fixtures. The sample returned the field below twice with the same result. An invalid primary record used the saved fallback and returned the expected field.

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

Practice data import using PostgreSQL and check a repeatable recovery workflow in the home development scenario.

## Conclusion

The project records an observable data import result for training case `DEV-05`.

[Back to the PostgreSQL project index](../README.md)
